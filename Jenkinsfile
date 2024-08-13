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
    ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ubuntu@${EC2_HOST} <<EOF
    # Hentikan dan hapus kontainer yang ada
    sudo docker stop ${CONTAINER_NAME} || true
    sudo docker rm ${CONTAINER_NAME} || true

    # Buat volume dan network jika belum ada
    sudo docker volume create dbstockbarang || true
    sudo docker network create stockbarang || true

    # Tarik dan jalankan kontainer
    sudo docker pull ${IMAGE_NAME}
    sudo docker run -d -p 3306:3306 --name dbstockbarang --restart unless-stopped -e MARIADB_ROOT_PASSWORD=${DBPASSWORD} -e MARIADB_DATABASE=stockbarang --network stockbarang -v dbstockbarang:/var/lib/mysql docker.io/mariadb
    sudo docker run -d -p 8080:80 -e PMA_HOST=dbstockbarang --name phpmyadminstockbarang --restart unless-stopped --network stockbarang docker.io/phpmyadmin
    sudo docker run -d --name ${CONTAINER_NAME} --network stockbarang -p 80:80 --restart unless-stopped ${IMAGE_NAME}
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
