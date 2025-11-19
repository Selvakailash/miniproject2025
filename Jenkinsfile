pipeline {
    agent any

    environment {
        BLUE_SERVER = "ubuntu@34.200.235.201"
        GREEN_SERVER = "ubuntu@18.208.207.12"
        WAR_FILE = "bluegreen-webapp/target/bluegreen-webapp.war"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/Selvakailash/miniproject2025.git', branch: 'master'
            }
        }

        stage('Build WAR') {
            steps {
                sh 'cd bluegreen-webapp && mvn clean package'
            }
        }

        stage('Deploy to BLUE Server') {
            steps {
                sh """
                scp -o StrictHostKeyChecking=no ${WAR_FILE} ${BLUE_SERVER}:/var/lib/tomcat9/webapps/
                """
            }
        }

        stage('Deploy to GREEN Server') {
            steps {
                sh """
                scp -o StrictHostKeyChecking=no ${WAR_FILE} ${GREEN_SERVER}:/var/lib/tomcat9/webapps/
                """
            }
        }
    }

    post {
        failure {
            echo "Deployment Failed!"
        }
        success {
            echo "Deployment Successful!"
        }
    }
}
