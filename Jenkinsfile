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
              scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/${env.JOB_NAME}/target/${env.JOB_NAME}.jar ec2-user@3.38.221.36:/home/ec2-user/${env.JOB_NAME}.jar
              ssh ec2-user@172.31.58.15 -o StrictHostKeyChecking=no

              mkdir ${env.JOB_NAME}
              cd ${env.JOB_NAME}
              mv /home/ec2-user/${env.JOB_NAME}.jar .

              java -jar ${env.JOB_NAME}.jar &
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

