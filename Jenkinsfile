pipeline {
  agent any

  tools {
    maven "maven"
  }

  environment {
    gitCredential = credentials('githubcd')
    ECR_PATH = '005040503934.dkr.ecr.ap-northeast-2.amazonaws.com'
    ECR_IMAGE = 'testcicd'
    REGION = 'ap-northeast-2'
    ACCOUNT_ID='005040503934'
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
            credentialsId: 'github',
            url: 'https://github.com/imyujinsim/testcicd.git'

            sh "ls -la"

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
            cp ./target/*.jar ./target/${ECR_IMAGE}.jar
            """
            env.mavenBuildResult=true
          } catch(error) {
            print('error')
            sh "rm -rf /var/lib/jenkins/workspace/*"
          }
        }
      }
    }

    stage('Deploy with ssh') {
        steps {
            try {
                sshagent(credentials: ['ssh-credential']) {
                    sh '''
                    scp -P 22 ./target/${ECR_IMAGE}.jar ec2-user@172.31.58.15:/home/ec2-user/${ECR_IMAGE}.jar
                    ssh -o StrictHostKeyChecking=no ${TARGET_HOST} "pwd"

                    ./mvnw package -Dskip-test             
                    '''
                }
            } catch(error) {
                print('error')
                sh "rm -rf /var/lib/jenkins/workspace/*"
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

