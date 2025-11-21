pipeline {
    agent any

    environment {
        BLUE_SERVER = "ubuntu@34.200.235.201"
        GREEN_SERVER = "ubuntu@18.208.207.12"

        WAR_FILE = "bluegreen-webapp/target/bluegreen-webapp.war"
        PEM_KEY = "/var/lib/jenkins/.ssh/miniproject.pem"
        TOMCAT_HOME = "/opt/tomcat11"

        LISTENER_ARN = "arn:aws:elasticloadbalancing:us-east-1:767124095373:listener/app/blue-green-alb/743cf5b73396d167/fdd30887c45394f6"
        BLUE_TG_ARN = "arn:aws:elasticloadbalancing:us-east-1:767124095373:targetgroup/blue-tg/9cad30252ccf28a0"
        GREEN_TG_ARN = "arn:aws:elasticloadbalancing:us-east-1:767124095373:targetgroup/green-tg/4186b8e09b21e0b9"
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
                    ${TOMCAT_HOME}/bin/shutdown.sh
                    rm -rf ${TOMCAT_HOME}/webapps/bluegreen-webapp*
                    mv /home/ubuntu/bluegreen-webapp.war ${TOMCAT_HOME}/webapps/
                    ${TOMCAT_HOME}/bin/startup.sh
                '
                """
            }
        }

        stage('Deploy to GREEN Server') {
            steps {
                sh """
                scp -i ${PEM_KEY} -o StrictHostKeyChecking=no ${WAR_FILE} ${GREEN_SERVER}:/home/ubuntu/
                ssh -i ${PEM_KEY} ${GREEN_SERVER} '
                    ${TOMCAT_HOME}/bin/shutdown.sh
                    rm -rf ${TOMCAT_HOME}/webapps/bluegreen-webapp*
                    mv /home/ubuntu/bluegreen-webapp.war ${TOMCAT_HOME}/webapps/
                    ${TOMCAT_HOME}/bin/startup.sh
                '
                """
            }
        }

        stage('Switch ALB Traffic to GREEN') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                    sh """
                    aws elbv2 modify-listener \
                        --listener-arn ${LISTENER_ARN} \
                        --default-actions Type=forward,TargetGroupArn=${GREEN_TG_ARN}
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Blue–Green Deployment Completed Successfully!"
        }
        failure {
            echo "Deployment Failed! Rolling back to BLUE"

            withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                sh """
                aws elbv2 modify-listener \
                    --listener-arn ${LISTENER_ARN} \
                    --default-actions Type=forward,TargetGroupArn=${BLUE_TG_ARN}
                """
            }
        }
    }
}
