pipeline{
    agent {label 'Jenkins-Agent' }
    tools {
        maven 'Maven3'
        jdk 'Java17'
    }
    environment {
        MAVEN_OPTS = '-Xmx1024m -XX:MaxPermSize=256m'
        APPLOCATION_NAME = "registration-app-aws"
        APPLOCATION_VERSION = "1.0.0"
        DOCKER_USER = "speedskillsx"
        DOCKER_PASS= "jenkins-docker-token"

        DOCKER_IMAGE = "${DOCKER_USER}/${APPLOCATION_NAME}"
        DOCKER_TAG = "${APPLOCATION_VERSION}-${BUILD_NUMBER}"
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
                    docker.withRegistry("", "DOCKER_PASS") {
                        def app = docker.build("${DOCKER_IMAGE}")
                    }

                    docker.withRegistry("", "DOCKER_PASS") {
                        docker.push("${DOCKER_TAG}")
                        docker.push("latest")    
                    }
                }
            }
        }
    }
}
