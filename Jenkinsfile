pipeline {
    agent any

    environment {
        BLUE_SERVER_IP = "34.200.235.201"
        GREEN_SERVER_IP = "18.208.207.12"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/Selvakailash/miniproject2025.git'
            }
        }

        stage('Build WAR') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Deploy to BLUE Server') {
            steps {
                sshagent(['tomcat-blue']) {
                    sh '''
                        echo "Deploying WAR to Blue Server..."
                        scp -o StrictHostKeyChecking=no target/*.war ubuntu@34.200.235.201:/opt/tomcat11/webapps/blue.war
                    '''
                }
            }
        }

        stage('Deploy to GREEN Server') {
            steps {
                sshagent(['tomcat-green']) {
                    sh '''
                        echo "Deploying WAR to Green Server..."
                        scp -o StrictHostKeyChecking=no target/*.war ubuntu@18.208.207.12:/opt/tomcat11/webapps/green.war
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Blue-Green Deployment Completed Successfully!"
        }
        failure {
            echo "Deployment Failed!"
        }
    }
}
