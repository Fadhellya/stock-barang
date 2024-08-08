pipeline {
  agent any
  stages {
    stage('Check Code') {
      steps {
        git(url: 'https://github.com/Fadhellya/stock-barang', branch: 'main')
      }
    }

    stage('Build Image') {
      steps {
        script {
          docker.withServer('tcp://docker:2376', '') {
            sh 'docker build -f stockbarang/Dockerfile .'
          }
        }

      }
    }

  }
}