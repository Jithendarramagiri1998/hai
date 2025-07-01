pipeline {
  agent { label 'agent' }

  stages {
    stage('Clone Code') {
      steps {
        git 'https://github.com/<your-username>/html-hello-app.git'
      }
    }

    stage('Deploy to Apache') {
      steps {
        sh '''
          sudo cp hai.html /var/www/html/index.html
        '''
      }
    }
  }
}
