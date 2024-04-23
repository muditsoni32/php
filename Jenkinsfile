pipeline {
    agent any // Use any available Jenkins agent

    stages {
        stage('Build Docker Image') {
            steps {
                // Build the Docker image using the provided Dockerfile
                script {
                    def image = docker.build('my-php-app:latest', '-f Dockerfile .')
                    sh "docker tag my-php-app:latest muditsoni32/my-php-app:latest"
                    image.push()
                }
            }
        }

        stage('Tag and Push Docker Image') {
            steps {
                // Tag and Push the Docker image to a Docker registry
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerregistry') {
                        docker.image('muditsoni32/my-php-app:latest').push()
                    }
                }
            }
        }

        stage('Deploy Docker Image') {
            steps {
                // SSH into the target server and run the Docker container
                script {
                    sshagent(credentials: ['mudit_key']) {
                        sh '''
                            ssh ec2-user@18.206.147.42 "docker pull muditsoni32/my-php-app:latest && \
                            docker run -d --name my-php-app -p 80:80 muditsoni32/my-php-app:latest"
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
