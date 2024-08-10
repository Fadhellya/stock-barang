pipeline {
    agent any

    environment {
        IMAGE_NAME = 'alexhermansyah/stockbarang:latest'
        CONTAINER_NAME = 'stockbarang_container'
        DOCKER_USERNAME = credentials('usernamedocker')
        DOCKER_PASSWORD = credentials('passworddocker')
        EC2_HOST = 'ubuntu@ec2-52-54-155-185.compute-1.amazonaws.com'  // Ganti dengan IP publik EC2
        SSH_CREDENTIALS_ID = 'ec2-ssh-key'  // Ganti dengan ID kredensial SSH yang ditambahkan ke Jenkins
    }

    options {
        timeout(time: 20, unit: 'MINUTES')
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
                    sh """
                    docker build -t ${IMAGE_NAME} .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sh """
                    docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
                    docker push ${IMAGE_NAME}
                    """
                }
            }
        }

        stage('Deploy Docker Container on EC2') {
            steps {
                script {
                    // SSH into EC2 and run Docker commands
                    sshagent(['${SSH_CREDENTIALS_ID}']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_HOST} << EOF
                        docker stop ${CONTAINER_NAME} || true
                        docker rm ${CONTAINER_NAME} || true
                        docker pull ${IMAGE_NAME}
                        docker run -d --name ${CONTAINER_NAME} -p 80:80 ${IMAGE_NAME}
                        EOF
                        """
                    }
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
