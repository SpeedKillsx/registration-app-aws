pipeline{
    agent {label 'Jenkins-Agent' }
    tools {
        maven 'Maven3'
        jdk 'Java17'
    }
    environment {
        APP_NAME = "registration-app-aws"
        RELEASE = "1.0.0"
        DOCKER_USER = "speedskillsx"
        DOCKER_PASS= 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}"+"/"+"${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }
    stages{
        stage("Clean Workspace") {
            steps {
                cleanWs()
            }
        }
        
        stage("Checkout Code") {
            steps {
                git branch: 'main', credentialsId: 'github', url: "https://github.com/SpeedKillsx/registration-app-aws"
            }
        }

        stage("Build Project") {
            steps {
                sh "mvn clean package"
            }
        }

        stage("Test Project"){
            steps{
                sh "mvn test"
            }
        }

        stage("Enable SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId:'jenkins-sonarqube-token') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    timeout(time: 10, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                    }
                }
            }
        }

        stage("Build & Push a Docker Image") {
            steps {
                script {
                    docker.withRegistry("", DOCKER_PASS) {
                        IMAGE_NAME = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry("", DOCKER_PASS) {
                        IMAGE_NAME.push("${IMAGE_TAG})")
                        IMAGE_NAME.push("latest")
                    }
                }
            }
        }
    }
}
