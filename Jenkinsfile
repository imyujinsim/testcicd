pipeline {
  agent any
  
  stages {
    stage('Git Clone from gitSCM') {
      steps {
        script {
          try {
            git branch: 'main'
            credentialsId: 'git'
            url: 'https://github.com/imyujinsim/testcicd'

            sh "ls -lat"
            print('github is succesfully connected!')
          } 
        }
      }
    }
  }
}
