pipeline {
  agent any

  tools {
    maven "maven"
  }

  stages {
    stage('Git Clone from gitSCM') {
      steps {
        script {
          try {
            sh """
            mkdir ${env.JOB_NAME}
            cd ${env.JOB_NAME}
            """
            git branch: 'main',
            credentialsId: 'imyujinsim',
            url: 'https://github.com/imyujinsim/testcicd.git'

            print('github is succesfully connected!')
          }
          catch (error) {
            sh "rm -rf /var/lib/jenkins/workspace/*"
            print('error')
          }
        }
      }
    }

    stage('Build JAR with Maven') {
      steps {
        script {
          try {
            sh """
            ./mvnw package -Dskip-test
            cp ./target/*.jar ./target/${env.JOB_NAME}.jar
            """

          } catch(error) {
            print('error')
            sh "rm -rf /var/lib/jenkins/workspace/*"
          }
        }
      }
    }

    stage('Deploy with ssh') {
      steps {
        script {
          try {
            sshagent(credentials: ['ssh-credential']) {
              sh """
              scp -P 22 ./target/${env.JOB_NAME}.jar ec2-user@172.31.58.15:/home/ec2-user/${env.JOB_NAME}.jar
              ssh -o StrictHostKeyChecking=no ${TARGET_HOST} "pwd"

              java -jar ${env.JOB_NAME}.jar 
              """
            }
          } catch(error) {
            print('error')
            sh "rm -rf /var/lib/jenkins/workspace/*"
          }
        }
      }
    }

    stage('Finish') {
      steps {
        sh "rm -rf /var/lib/jenkins/workspace/*"
      }
    }
  }
}

