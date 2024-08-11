pipeline {
  agent any
  environment {
    IMAGE_NAME = 'alexhermansyah/stockbarang:latest'
    CONTAINER_NAME = 'stockbarang_container'
    DOCKER_CREDENTIALS = credentials('dockerhub-credentials')
    EC2_HOST = '52.54.155.185'
    SSH_CREDENTIALS_ID = credentials('ec2-ssh')
  }
  stages {
    stage('Checkout') {
      steps {
        git(url: 'https://github.com/Fadhellya/stock-barang.git', branch: 'main', credentialsId: 'github-login')
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          sh 'docker build -t ${IMAGE_NAME} .'
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        script {
          sh '''
          echo "${DOCKER_CREDENTIALS_PSW}" | docker login -u "${DOCKER_CREDENTIALS_USR}" --password-stdin
          docker push ${IMAGE_NAME}
          '''
        }
      }
    }

    stage('Deploy Docker Container on EC2') {
      steps {
        script {
          sshagent([SSH_CREDENTIALS_ID]) {
            sh '''
            ssh -o StrictHostKeyChecking=no ubuntu@${EC2_HOST} << EOF
            sudo docker stop ${CONTAINER_NAME} || true
            sudo docker rm ${CONTAINER_NAME} || true
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
  options {
    timeout(time: 20, unit: 'MINUTES')
  }
}
