pipeline {
  agent any

  tools {
    maven "maven"
    jdk 'java'
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
              scp -o StrictHostKeyChecking=no ./target/${env.JOB_NAME}.jar ec2-user@3.38.221.36:~/${env.JOB_NAME}.jar
              ssh ec2-user@172.31.58.15 -o StrictHostKeyChecking=no
              """

              sh '''
              #!/bin/bash 
              kill -9 $(lsof -t -i:8080)
              '''

              sh """
              java -jar ${env.JOB_NAME}.jar %
              """
            }
          } catch(error) {
            print('error')
            sh "rm -rf /home/ec2-user/*.jar"
          }
        }
      }
    }

    stage('Finish') {
      steps {
        sh "rm -rf /home/ec2-user/*.jar"
        sh "exit"
        sh "rm -rf /var/lib/jenkins/workspace/*"
      }
    }
  }
}

