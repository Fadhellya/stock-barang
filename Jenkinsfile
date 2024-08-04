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
        sh 'docker build -f stockbarang/stockbarang/Dockerfile .'
      }
    }

  }
}