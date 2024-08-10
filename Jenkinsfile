pipeline {
    agent any

    environment {
        IMAGE_NAME = 'alexhermansyah/stockbarang:latest'
        CONTAINER_NAME = 'stockbarang_container'
    }

    options {
        timeout(time: 20, unit: 'MINUTES') // Custom timeout
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image
                    sh """
                    docker build -t ${IMAGE_NAME} .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Push Docker image to Docker Hub
                    sh """
                    docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
                    docker push ${IMAGE_NAME}
                    """
                }
            }
        }

        stage('Deploy Docker Container') {
            steps {
                script {
                    // Stop the existing container if it exists
                    sh """
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                    """

                    // Run the Docker container
                    sh """
                    docker run -d --name ${CONTAINER_NAME} -p 80:80 ${IMAGE_NAME}
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'Deployment succeeded!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
