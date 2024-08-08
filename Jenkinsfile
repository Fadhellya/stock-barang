pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps {
        git(branch: 'main', url: 'https://github.com/Fadhellya/stock-barang')
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          docker.build("${IMAGE_NAME}:${env.BUILD_ID}")
        }

      }
    }

    stage('Push Docker Image') {
      steps {
        script {
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
  environment {
    IMAGE_NAME = 'alexhermansyah/stockbarang'
    DOCKER_REGISTRY = 'docker.io'
    DOCKER_CREDENTIALS = '1f149eee-f169-4965-bf2e-468ddafdb59a'
  }
  post {
    always {
      sh '''
            docker stop php-native-app || true
            docker rm php-native-app || true
            docker rmi ${IMAGE_NAME}:${env.BUILD_ID} || true
            '''
    }

  }
}