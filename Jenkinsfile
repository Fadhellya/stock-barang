pipeline {
    agent any

    environment {
        IMAGE_NAME = 'alexhermansyah/stockbarang:latest'
        CONTAINER_NAME = 'stockbarang_container'
        DB_CONTAINER_NAME = 'dbstockbarang'
        DB_VOLUME_NAME = 'dbstockbarang_volume'
        DB_NETWORK_NAME = 'stockbarang_network'
        PHPMYADMIN_CONTAINER_NAME = 'phpmyadmin_stockbarang'
        DOCKER_USERNAME = credentials('usernamedocker')
        DOCKER_PASSWORD = credentials('passworddocker')
        EC2_HOST = '52.54.155.185'
        DBPASSWORD = credentials('dbpassword')
        SSH_KEY_ID = 'remote-ec2-ssh'
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
                        ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ubuntu@${EC2_HOST} << 'EOF'
                        
                        # Create Docker Volume if it doesn't exist
                        sudo docker volume ls | grep -q ${DB_VOLUME_NAME} || sudo docker volume create ${DB_VOLUME_NAME}
                        
                        # Create Docker Network if it doesn't exist
                        sudo docker network ls | grep -q ${DB_NETWORK_NAME} || sudo docker network create ${DB_NETWORK_NAME}
                        
                        # Stop and Remove existing containers if they exist
                        sudo docker stop ${DB_CONTAINER_NAME} || true
                        sudo docker rm ${DB_CONTAINER_NAME} || true
                        sudo docker stop ${PHPMYADMIN_CONTAINER_NAME} || true
                        sudo docker rm ${PHPMYADMIN_CONTAINER_NAME} || true
                        sudo docker stop ${CONTAINER_NAME} || true
                        sudo docker rm ${CONTAINER_NAME} || true
                        
                        # Run Database Container
                        sudo docker run -d -p 3306:3306 --name ${DB_CONTAINER_NAME} --restart unless-stopped \
                        -e MARIADB_ROOT_PASSWORD=${DBPASSWORD} -e MARIADB_DATABASE=stockbarang \
                        --network ${DB_NETWORK_NAME} -v ${DB_VOLUME_NAME}:/var/lib/mysql mariadb:latest
                        
                        # Run phpMyAdmin Container
                        sudo docker run -d -p 8080:80 -e PMA_HOST=${DB_CONTAINER_NAME} \
                        --name ${PHPMYADMIN_CONTAINER_NAME} --restart unless-stopped \
                        --network ${DB_NETWORK_NAME} phpmyadmin:latest
                        
                        # Ensure Docker Login
                        sudo docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}
                        
                        # Pull the Latest Image
                        sudo docker pull ${IMAGE_NAME}
                        
                        # Run Application Container
                        sudo docker run -d --name ${CONTAINER_NAME} --network ${DB_NETWORK_NAME} \
                        -p 80:80 --restart unless-stopped ${IMAGE_NAME}
                        
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
