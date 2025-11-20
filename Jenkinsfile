pipeline {
    agent any

    environment {
        BLUE_SERVER = "ubuntu@34.200.235.201"
        GREEN_SERVER = "ubuntu@18.208.207.12"
        WAR_FILE = "bluegreen-webapp/target/bluegreen-webapp.war"
        PEM_KEY = "/var/lib/jenkins/.ssh/miniproject.pem"
        TOMCAT_PATH = "/opt/tomcat11/webapps"
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
                scp -i ${PEM_KEY} -o StrictHostKeyChecking=no ${WAR_FILE} ${BLUE_SERVER}:/home/ubuntu/
                ssh -i ${PEM_KEY} ${BLUE_SERVER} '
                    sudo mv /home/ubuntu/bluegreen-webapp.war ${TOMCAT_PATH}/ &&
                    sudo chown -R ubuntu:ubuntu ${TOMCAT_PATH}/bluegreen-webapp.war &&
                    sudo systemctl restart tomcat11
                '
                """
            }
        }

        stage('Deploy to GREEN Server') {
            steps {
                sh """
                scp -i ${PEM_KEY} -o StrictHostKeyChecking=no ${WAR_FILE} ${GREEN_SERVER}:/home/ubuntu/
                ssh -i ${PEM_KEY} ${GREEN_SERVER} '
                    sudo mv /home/ubuntu/bluegreen-webapp.war ${TOMCAT_PATH}/ &&
                    sudo chown -R ubuntu:ubuntu ${TOMCAT_PATH}/bluegreen-webapp.war &&
                    sudo systemctl restart tomcat11
                '
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
