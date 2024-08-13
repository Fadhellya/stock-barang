pipeline {
    agent any

    environment {
        IMAGE_NAME = 'alexhermansyah/stockbarang:latest'
        CONTAINER_NAME = 'stockbarang_container'
        DOCKER_USERNAME = credentials('usernamedocker')
        DOCKER_PASSWORD = credentials('passworddocker')
        EC2_HOST = '52.54.155.185'
        SSH_KEY_ID = 'remote-ec2-ssh'  // Ganti dengan ID secret file yang mengandung private key
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
                    sh '''
                    docker build -t ${IMAGE_NAME} .
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sh '''
                    docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}
                    docker push ${IMAGE_NAME}
                    '''
                }
            }
        }

        stage('Deploy Docker Container on EC2') {
            steps {
                script {
                    echo "Deploying Docker Container on EC2"
                    echo "EC2 Host: ${EC2_HOST}"
                    withCredentials([file(credentialsId: "${SSH_KEY_ID}", variable: 'SSH_KEY')]) {
                        sh '''
                        ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ubuntu@${EC2_HOST} << EOF
                        sudo docker stop ${CONTAINER_NAME} || true
                        sudo docker rm ${CONTAINER_NAME} || true
                        sudo docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}
                        sudo docker pull ${IMAGE_NAME}
                        sudo docker run -d --name ${CONTAINER_NAME} -p 80:80 --restart unless-stopped ${IMAGE_NAME}
                        EOF
                        '''
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
