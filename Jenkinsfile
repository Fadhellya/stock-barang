pipeline {
    agent any

    environment {
        IMAGE_NAME = 'alexhermansyah/stockbarang:latest'
        CONTAINER_NAME = 'stockbarang_container'
        DOCKER_USERNAME = credentials('usernamedocker')
        DOCKER_PASSWORD = credentials('passworddocker')
        EC2_HOST = 'ec2-52-54-155-185.compute-1.amazonaws.com'
        SSH_CREDENTIALS_ID = 'ssh-agent-jenkins'  // Pastikan SSH_CREDENTIALS_ID ini sesuai dengan ID yang ada di Jenkins
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
                    sshagent(['ssh-agent-jenkins']) {
                        sh '''
                        ssh -o StrictHostKeyChecking=no 'ec2-52-54-155-185.compute-1.amazonaws.com' << EOF
                        sudo docker stop 'stockbarang_container' || true
                        sudo docker rm 'stockbarang_container' || true
                        sudo docker pull 'alexhermansyah/stockbarang:latest'
                        sudo docker run -d --name 'stockbarang_container' -p 80:80 --restart unless-stopped 'alexhermansyah/stockbarang:latest'
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
