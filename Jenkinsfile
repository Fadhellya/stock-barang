pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // Pull the latest code from your Git repository
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image
                    def dockerImage = docker.build("alexhermansyah/stockbarang:latest")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', 'a3dce63d-f031-4509-a376-739a32c9ec4a') {
                        // Push Docker image to Docker Hub
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy Docker Container') {
            steps {
                script {
                    // Stop and remove existing container if it exists
                    sh '''
                    docker stop php-app || true
                    docker rm php-app || true
                    '''

                    // Run a new container with the latest image
                    sh '''
                    docker run -d --name php-app -p 80:80 alexhermansyah/stockbarang:latest
                    '''
                }
            }
        }
    }

    post {
        always {
            // Cleanup workspace after pipeline execution
            cleanWs()
        }
    }
}
