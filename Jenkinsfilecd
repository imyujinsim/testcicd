pipeline {
  agent any

  tools {
    maven "Maven"
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

    stage('Docker Build and Push to ECR') {
      steps {
        script {
          try {
            sh """
            #!/bin/bash
            cat > test << EOF
            FROM openjdk:17-alpine
            ADD ./target/${ECR_IMAGE}.jar /home/${ECR_IMAGE}.jar
            CMD ["nohup", "java", "-jar", "-Dspring.profiles.active='mysql'", "/home/${ECR_IMAGE}.jar"]
            """
            sh "mv test Dockerfile"

            docker.withRegistry("https://${ECR_PATH}", "ecr:ap-northeast-2:aws_credentials") {
              def image = docker.build("${ECR_PATH}/${ECR_IMAGE}:${env.BUILD_NUMBER}")
              image.push()
            }
          }
          catch(error) {
            print(error)
            echo 'Remove Deploy Files'
            sh "rm -rf /var/lib/jenkins/workspace/*"
          }
        }
      }
    }

    stage('Push Yaml'){
      steps {
        git url: 'https://github.com/imyujinsim/testcicd-cd.git', branch: "main", credentialsId: 'githubicd'
        
        sh """
        #!/bin/bash
        cat > deploy.yaml << EOF
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: tomcat-deployment
        spec:
          replicas: 2
          selector:
            matchLabels:
              app: was
          template:
            metadata:
              labels:
                app: was
            spec:
              containers:
              - image: 005040503934.dkr.ecr.ap-northeast-2.amazonaws.com/${ECR_IMAGE}:${env.BUILD_NUMBER}
                name: petclinic
                ports:
                - name: tcp
                  containerPort: 8080
        """
 
        sh '''
        git add deploy.yaml
        git commit -m 'yaml for deploy'
        git push https://${gitCredential}@github.com/imyujinsim/testcicd-cd.git main
        '''
      }
    }

    stage('Finish') {
      steps {
        sh "rm -rf /var/lib/jenkins/workspace/*"
      }
    }
  }
}

