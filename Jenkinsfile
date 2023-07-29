pipeline {
  agent any

  stages {
    stage('Git Clone from gitSCM') {
      steps {
        script {
          try {
            git branch: 'main'
            credentialsId: 'github'
            url: 'https://github.com/imyujinsim/testcicd'

            sh "ls -lat"
            print('github is succesfully connected!')
          }
          catch(error) {
            print('error:')
          }
        }
      }
    }
  }
}
