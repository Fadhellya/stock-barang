pipeline {
    agent any

    environment {
        IMAGE_NAME = 'alexhermansyah/stockbarang'
        DOCKER_REGISTRY = 'docker.io'  // Ganti dengan registry Docker Anda jika diperlukan
        DOCKER_CREDENTIALS = '1f149eee-f169-4965-bf2e-468ddafdb59a'  // Ganti dengan ID kredensial Docker Anda di Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout kode dari repository
                git branch: 'main', url: 'https://github.com/Fadhellya/stock-barang'  // Ganti dengan URL repository Anda
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image
                    docker.build("${IMAGE_NAME}:${env.BUILD_ID}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Login ke Docker registry
                    docker.withRegistry("https://${DOCKER_REGISTRY}", "${DOCKER_CREDENTIALS}") {
                        // Push Docker image ke registry
                        docker.image("${IMAGE_NAME}:${env.BUILD_ID}").push()
                    }
                }
            }
        }

        stage('Deploy Docker Container') {
            steps {
                script {
                    // Jalankan container dari Docker image yang telah dibangun
                    sh '''
                    docker run -d \
                        --name alexhermansyah/stockbarang \
                        -p 80:80 \
                        --env DB_HOST=${DB_HOST} \
                        --env DB_USER=${DB_USER} \
                        --env DB_PASSWORD=${DB_PASSWORD} \
                        --env DB_NAME=${DB_NAME} \
                        ${IMAGE_NAME}:${env.BUILD_ID}
                    '''
                }
            }
        }
    }

    post {
        always {
            // Cleanup: Hentikan dan hapus container, hapus image yang dibuat
            sh '''
            docker stop php-native-app || true
            docker rm php-native-app || true
            docker rmi ${IMAGE_NAME}:${env.BUILD_ID} || true
            '''
        }
    }
}
