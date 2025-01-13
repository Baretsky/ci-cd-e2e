pipeline{
    agent{
        label "jenkins-agent"
    }

    tools{
        jdk 'java17'
        maven 'Maven3'
    }

    environment{
        APP_NAME = "ci-cd-e2e"
        RELEASE = "1.0.0"
        DOCKER_USER = "baretsky24"
        DOCKER_PASS = "dockerhub"
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials('JENKINS_API_TOKEN')
    }

    stages{
        stage("Cleanup Workspace"){
            steps{
                cleanWs()
            }
        }

        stage("Checkout from SCM"){
            steps{
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Baretsky/ci-cd-e2e'
            }
        }

        stage("Build Application"){
            steps{
                sh "mvn clean package"
            }
        }

        stage("Test Application"){
            steps{
                sh "mvn test"
            }
        }
        
        stage("Sonarqube Analysis"){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token'){
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }

        stage("Quality Gate"){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }
            }
        }

        stage("Build & Push Docker Image"){
            steps{
                script{
                    docker.withRegistry('', DOCKER_PASS){
                        docker_image = docker.build("${IMAGE_NAME}")
                    }

                    docker.withRegistry('', DOCKER_PASS){
                        docker_image.push("${IMAGE_TAG}")
                    }
                }
            }
        }

        stage("Trigger CD Pipeline"){
            steps{
                script{
                    sh "curl -v -k --user baretsky:${JENKINS_API_TOKEN} -X POST -H 'cache-conrtol: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'https://jenkins.baretsky.net/job/ci-cd-e2e-deployment/buildWithParameters?token=gitops-token'"
                }
            }
        }
    }
}