pipeline {
    agent any // Use any available Jenkins agent

    environment {
        DOCKER_CREDENTIALS_ID = 'dockerregistry' // ID of Jenkins credentials for Docker Hub
        REMOTE_HOST = '54.158.234.194' // Remote server where Docker build will take place
        REMOTE_USER = 'ec2-user' // Assuming it's an EC2 instance, typically 'ec2-user'
    }

    stages {
        stage('Build and Tag Docker Image on Remote') {
            steps {
                script {
                    sshagent(credentials: ['mudit_key']) {
                        // Execute Docker build and tag commands remotely
                        sh '''
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} \
                            "docker build -t my-php-app:latest -f Dockerfile . && \
                            docker tag my-php-app:latest muditsoni32/my-php-app:latest"
                        '''
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sshagent(credentials: ['mudit_key']) {
                        // Push the Docker image from remote server to Docker Hub
                        sh '''
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} \
                            "docker login -u muditsoni32 -p [PASSWORD] && \
                            docker push muditsoni32/my-php-app:latest"
                        '''
                    }
                }
            }
        }

        stage('Deploy Docker Image') {
            steps {
                script {
                    sshagent(credentials: ['mudit_key']) {
                        // SSH into the target server and run the Docker container
                        sh '''
                        ssh -o StrictHostKeyChecking=no ec2-user@18.206.147.42 \
                            "sudo docker pull muditsoni32/my-php-app:latest && \
                            sudo docker run -d --name my-php-app -p 80:80 muditsoni32/my-php-app:latest"
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed. Check the console output for errors.'
        }
    }
}
