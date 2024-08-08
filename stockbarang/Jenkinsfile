pipeline {
    agent any

    environment {
        IMAGE_NAME = 'alexhermansyah/stockbarang'
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_CREDENTIALS = '1f149eee-f169-4965-bf2e-468ddafdb59a'
        REMOTE_HOST = 'ubuntu@ec2-3-212-107-115.compute-1.amazonaws.com' // Ganti dengan user dan IP EC2 Anda
        SSH_CREDENTIALS = '7edbb822-7f70-411e-9638-14d7ec423781' // Ganti dengan ID kredensial SSH di Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Fadhellya/stock-barang'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // SSH ke EC2 dan build Docker image
                    sshagent([SSH_CREDENTIALS]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_HOST} << EOF
                            docker build -t ${IMAGE_NAME}:${env.BUILD_ID} .
                        EOF
                        """
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sshagent([SSH_CREDENTIALS]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_HOST} << EOF
                            docker login -u \$DOCKER_USERNAME -p \$DOCKER_PASSWORD ${DOCKER_REGISTRY}
                            docker push ${IMAGE_NAME}:${env.BUILD_ID}
                        EOF
                        """
                    }
                }
            }
        }

        stage('Deploy Docker Container') {
            steps {
                script {
                    sshagent([SSH_CREDENTIALS]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_HOST} << EOF
                            docker run -d --name stockbarang-container -p 80:80 \
                            --env DB_HOST=${DB_HOST} \
                            --env DB_USER=${DB_USER} \
                            --env DB_PASSWORD=${DB_PASSWORD} \
                            --env DB_NAME=${DB_NAME} \
                            ${IMAGE_NAME}:${env.BUILD_ID}
                        EOF
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                sshagent([SSH_CREDENTIALS]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${REMOTE_HOST} << EOF
                        docker stop stockbarang-container || true
                        docker rm stockbarang-container || true
                        docker rmi ${IMAGE_NAME}:${env.BUILD_ID} || true
                    EOF
                    """
                }
            }
        }
    }
}
