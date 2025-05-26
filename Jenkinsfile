pipeline {
    agent { label 'Jenkins-Agent' }

    tools {
        maven 'Maven3'
        jdk 'Java17'
    }

    environment {
        APP_NAME = "registration-app-aws"
        RELEASE = "1.0.0"
        DOCKER_USER = "speedskillsx"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
    }

    stages {
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
                sh "echo 'Listing target directory contents:'"
                sh "ls -lh **/target"
            }
        }

        stage("Test Project") {
            steps {
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
                    sh "echo 'Building Docker image: ${IMAGE_NAME}:${IMAGE_TAG}'"
                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}:${IMAGE_TAG}"
                    }

                    sh "echo 'Pushing Docker image with tag: ${IMAGE_TAG}'"
                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push("latest")
                    }

                    sh "echo 'Docker image pushed successfully: ${IMAGE_NAME}:${IMAGE_TAG}'"
                }
            }
        }

        stage("Trivy Scan") {
            steps {
                script {
                    sh ("docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image ${IMAGE_NAME}:${IMAGE_TAG} --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table")
                }
            }
        }

        stage("Clean Artifacts") {
            steps {
                sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                sh "docker rmi ${IMAGE_NAME}:latest"
            }
        }

        stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user clouduser:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-13-39-79-153.eu-west-3.compute.amazonaws.com:8080/job/gitops-registration-app-cd/buildWithParameters?token=gitops-token'"
                }
            }
       }
    
    }
}
