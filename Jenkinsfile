pipeline {
  agent any

  tools {
    maven "Maven"
  }

  environment {
    registryCredential = 'aws_credential'
    ECR_PATH = '005040503934.dkr.ecr.ap-northeast-2.amazonaws.com/testcicd'
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

    stage('Docker Build and Push to ECR') {
      steps {
        script {
          try {
            sh """
	    #!/bin/bash
            cat > Dockerfile <<-EOF
            FROM openjdk:11-jre-slim
            ADD ./target/${ECR_IMAGE}.jar /home/${ECR_IMAGE}.jar
            CMD ["nohup", "java", "-jar", "-Dspring.profiles.active='mysql'", "/home/${ECR_IMAGE}.jar"]
            """
	    sh "EOF"
            docker.withRegistry("https://${ECR_PATH}") {
              def image = docker.build("${ECR_PATH}/${ECR_IMAGE}:${env.BUILD_NUMBER}")
              image.push()
            }
            echo 'Remove Deploy Files'
            sh "rm -rf /var/lb/jenkins/*"
            env.dockerBuildResult=true
          }
          catch(error) {
            print(error)
              echo 'Remove Deploy Files'
              sh "rm -rf /var/lib/jenkins/workspace/*"
          }
        }
      }
    }
  }
}
