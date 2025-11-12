pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/Selvakailash/miniproject2025.git'
            }
        }

        stage('Deploy to Blue Server') {
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: 'blue-server',
                            transfers: [
                                sshTransfer(
                                    sourceFiles: 'index.html',
                                    remoteDirectory: '/var/www/html',
                                    execCommand: '''
                                        sudo rm -rf /var/www/html
                                        sudo mv /var/www/index.html /var/www/html
                                        sudo systemctl restart apache2
                                    '''
                                )
                            ]
                        )
                    ]
                )
            }
        }

        stage('Deploy to Green Server') {
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: 'green-server',
                            transfers: [
                                sshTransfer(
                                    sourceFiles: 'index.html',
                                    remoteDirectory: '/var/www/html',
                                    execCommand: '''
                                        sudo rm -rf /var/www/html
                                        sudo mv /var/www/index.html /var/www/html
                                        sudo systemctl restart apache2
                                    '''
                                )
                            ]
                        )
                    ]
                )
            }
        }
    }
}
