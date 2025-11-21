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

        stage('Determine Live Server') {
            steps {
                script {
                    withAWS(credentials: 'AWS', region: 'us-east-1') {
                        LIVE_TG_ARN = sh(script: """aws elbv2 describe-listeners \
                            --listener-arn ${LISTENER_ARN} \
                            --query "Listeners[0].DefaultActions[0].TargetGroupArn" \
                            --output text""", returnStdout: true).trim()
                    }

                    if (LIVE_TG_ARN == BLUE_TG_ARN) {
                        LIVE_SERVER = BLUE_SERVER
                        IDLE_SERVER = GREEN_SERVER
                        IDLE_TG_ARN = GREEN_TG_ARN
                    } else {
                        LIVE_SERVER = GREEN_SERVER
                        IDLE_SERVER = BLUE_SERVER
                        IDLE_TG_ARN = BLUE_TG_ARN
                    }

                    echo "Live Server: ${LIVE_SERVER}"
                    echo "Idle Server: ${IDLE_SERVER}"
                }
            }
        }

        stage('Deploy to Idle Server') {
            steps {
                sh """
                scp -i ${PEM_KEY} -o StrictHostKeyChecking=no ${WAR_FILE} ${IDLE_SERVER}:/home/ubuntu/
                ssh -i ${PEM_KEY} ${IDLE_SERVER} '
                    ${TOMCAT_HOME}/bin/shutdown.sh
                    rm -rf ${TOMCAT_HOME}/webapps/bluegreen-webapp*
                    mv /home/ubuntu/bluegreen-webapp.war ${TOMCAT_HOME}/webapps/
                    ${TOMCAT_HOME}/bin/startup.sh
                '
                """
            }
        }

        stage('Switch ALB Traffic to Idle Server') {
            steps {
                withAWS(credentials: 'aws', region: 'us-east-1') {
                    sh """
                    aws elbv2 modify-listener \
                        --listener-arn ${LISTENER_ARN} \
                        --default-actions Type=forward,TargetGroupArn=${IDLE_TG_ARN}
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
            echo "Deployment Failed! Rolling back to previous live server."

            withAWS(credentials: 'aws', region: 'us-east-1') {
                sh """
                aws elbv2 modify-listener \
                    --listener-arn ${LISTENER_ARN} \
                    --default-actions Type=forward,TargetGroupArn=${LIVE_TG_ARN}
                """
            }
        }
    }
}
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

        stage('Determine Live Server') {
            steps {
                script {
                    // Query ALB to find current live target group
                    def listener = sh(script: """
                        aws elbv2 describe-listeners --listener-arn ${LISTENER_ARN} \
                        --query 'Listeners[0].DefaultActions[0].ForwardConfig.TargetGroups[0].TargetGroupArn' \
                        --output text
                    """, returnStdout: true).trim()

                    if (listener == BLUE_TG_ARN) {
                        env.LIVE_SERVER = "BLUE"
                        env.IDLE_SERVER = "GREEN"
                    } else {
                        env.LIVE_SERVER = "GREEN"
                        env.IDLE_SERVER = "BLUE"
                    }
                    echo "Live Server: ${env.LIVE_SERVER}, Idle Server: ${env.IDLE_SERVER}"
                }
            }
        }

        stage('Deploy to Idle Server') {
            steps {
                script {
                    def targetServer = (env.IDLE_SERVER == "BLUE") ? BLUE_SERVER : GREEN_SERVER

                    sh """
                        scp -i ${PEM_KEY} -o StrictHostKeyChecking=no ${WAR_FILE} ${targetServer}:/home/ubuntu/
                        ssh -i ${PEM_KEY} ${targetServer} '
                            ${TOMCAT_HOME}/bin/shutdown.sh
                            rm -rf ${TOMCAT_HOME}/webapps/bluegreen-webapp*
                            mv /home/ubuntu/bluegreen-webapp.war ${TOMCAT_HOME}/webapps/
                            ${TOMCAT_HOME}/bin/startup.sh
                        '
                    """
                }
            }
        }

        stage('Switch ALB Traffic to Idle Server') {
            steps {
                script {
                    def targetTG = (env.IDLE_SERVER == "BLUE") ? BLUE_TG_ARN : GREEN_TG_ARN

                    withAWS(credentials: 'AWS', region: 'us-east-1') {
                        sh """
                        aws elbv2 modify-listener \
                            --listener-arn ${LISTENER_ARN} \
                            --default-actions Type=forward,TargetGroupArn=${targetTG}
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Blue–Green Deployment Completed Successfully! New live server: ${env.IDLE_SERVER}"
        }
        failure {
            echo "Deployment Failed! Rolling back to previous live server: ${env.LIVE_SERVER}"

            script {
                def rollbackTG = (env.LIVE_SERVER == "BLUE") ? BLUE_TG_ARN : GREEN_TG_ARN
                withAWS(credentials: 'AWS', region: 'us-east-1') {
                    sh """
                    aws elbv2 modify-listener \
                        --listener-arn ${LISTENER_ARN} \
                        --default-actions Type=forward,TargetGroupArn=${rollbackTG}
                    """
                }
            }
        }
    }
}
