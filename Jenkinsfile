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
                    // Set a custom timeout (e.g., 10 minutes) for the Docker build process
                    timeout(time: 10, unit: 'MINUTES') {
                        def dockerImage = docker.build("alexhermansyah/stockbarang:latest")
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Set a custom timeout (e.g., 5 minutes) for pushing the image to Docker Hub
                    timeout(time: 5, unit: 'MINUTES') {
                        docker.withRegistry('', 'a3dce63d-f031-4509-a376-739a32c9ec4a') {
                            // Push Docker image to Docker Hub
                            dockerImage.push()
                        }
                    }
                }
            }
        }

        stage('Deploy Docker Container') {
            steps {
                script {
                    // Set a custom timeout (e.g., 5 minutes) for stopping, removing, and deploying the container
                    timeout(time: 5, unit: 'MINUTES') {
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
    }

    post {
        always {
            // Cleanup workspace after pipeline execution
            cleanWs()
        }
    }
}
