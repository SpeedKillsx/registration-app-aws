pipeline{
    agent {label 'Jenkins-Agent' }
    tools {
        maven 'Maven3'
        jdk 'Java17'
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
    }
}
