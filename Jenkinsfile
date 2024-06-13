pipeline {
    agent {
        label 'runthisonmyworkermachine'
    }
    environment {
        DOCKER_CREDENTIALS_ID = 'dockerregistry'
        REMOTE_HOST = '35.173.129.16'
        REMOTE_USER = 'ec2-user'
    }

    stages {
        stage('Prepare Remote Environment') {
            steps {
                script {
                    sshagent(credentials: ['mudit_key']) {
                        // Copying the Dockerfile and project files to the remote server
                        sh "rsync -avz --exclude '.git' -e 'ssh -o StrictHostKeyChecking=no' . ${REMOTE_USER}@${REMOTE_HOST}:/home/ec2-user/"
                    }
                }
            }
        }

        stage('Build and Tag Docker Image on Remote') {
            steps {
                script {
                    sshagent(credentials: ['mudit_key']) {
                        // Execute Docker build and tag commands remotely
                        sh '''
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} \
                            "cd /home/ec2-user/ && \
                             sudo docker build -t my-php-app:latest -f Dockerfile . && \
                             sudo docker tag my-php-app:latest muditsoni32/my-php-app:latest"
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
                            "docker login -u muditsoni32 -p mudit#@12 && \
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
                        ssh -o StrictHostKeyChecking=no ec2-user@3.80.30.99 \
                            "sudo docker stop my-php-app1 || true && \
                            sudo docker rm my-php-app1 || true && \
                            sudo docker rmi muditsoni32/my-php-app:latest || true && \
                            sudo docker pull muditsoni32/my-php-app:latest && \
                            sudo docker run -d --name my-php-app1 -p 80:80 muditsoni32/my-php-app:latest"
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
