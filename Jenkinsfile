pipeline {
  agent any

  tools {
    maven "Maven"
  }

  environment {
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
            git branch: 'main'
            url: 'https://github.com/imyujinsim/testcicd'

            env.cloneResult=true
            print('github is succesfully connected!')
          }
          catch (error) {
            sh "sudo rm -rf /var/lb/jenkins/*"
            print('error')
            env.cloneResult=false
            currentBuild.result = 'FAILURE'
          }
        }
      }
    }

    stage('Build JAR with Maven') {
      when {
        expression {
          return env.cloneResult ==~ /(?i)(Y|YES|T|TRUE|ON|RUN)/
        }
      }
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
            sh "sudo rm -rf /var/lb/jenkins/*"
            env.cloneResult=false
            currentBuild.result = 'FAILURE'
          }
        }
      }
    }

    stage('Docker Build and Push to ECR') {
      when {
        expression {
          return env.mavenBuildResult ==~ /(?i)(Y|YES|T|TRUE|ON|RUN)/
        }
      }
      steps {
        script {
          try {
            sh """
            cat > Dockerfile <<-EOF
            FROM openjdk:11-jre-slim
            ADD ./target/${ECR_IMAGE}.jar /home/${ECR_IMAGE}.jar
            CMD ["nohup", "java", "-jar", "-Dspring.profiles.active='mysql'", "/home/${ECR_IMAGE}.jar"]
            EOF
            """
            docker.withRegistry("https://${ECR_PATH}", "ecr:ap-northeast-2:aws_credentials") {
              def image = docker.build("${ECR_PATH}/${ECR_IMAGE}:${env.BUILD_NUMBER}")
              image.push()
            }
            echo 'Remove Deploy Files'
            sh "sudo rm -rf /var/lb/jenkins/*"
            env.dockerBuildResult=true
          }
          catch(error) {
            print(error)
              echo 'Remove Deploy Files'
              sh "sudo rm -rf /var/lb/jenkins/*"
              env.dockerBuildResult=false
              currentBuild.result = 'FAILURE'
          }
        }
      }
    }
  }
}
