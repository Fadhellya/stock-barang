pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps {
        checkout scm
        git(url: 'https://github.com/Fadhellya/stock-barang.git', branch: 'main', credentialsId: 'github-login')
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
          sshagent(['1f4c09fc-19ee-434c-94fa-c544bc7699f6']) {
            sh '''
ssh -o StrictHostKeyChecking=no '52.54.155.185' << EOF
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
  environment {
    IMAGE_NAME = 'alexhermansyah/stockbarang:latest'
    CONTAINER_NAME = 'stockbarang_container'
    DOCKER_USERNAME = credentials('usernamedocker')
    DOCKER_PASSWORD = credentials('passworddocker')
    EC2_HOST = '52.54.155.185'
    SSH_CREDENTIALS_ID = '1f4c09fc-19ee-434c-94fa-c544bc7699f6'
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