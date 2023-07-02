pipeline {
  agent {
    label 'worker'
  }
   
  stages {
    stage('Git Checkout') {
      steps {
        checkout([$class: 'GitSCM',
                  branches: [[name: '*/main']],
                  userRemoteConfigs: [[url: 'https://github.com/Kavyanjana/project-c42-kavya-jenkins.git']]])
      }
    }
    
     stage('Stopping the container') {
      steps {
        script {
          sh 'docker kill $(docker ps -q)'
        }
      }
    }
    stage('Build Docker Image') {
      steps {
        script {     
          sh '''
          aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 191351159627.dkr.ecr.us-east-1.amazonaws.com
          docker build -t final-assignment:${BUILD_NUMBER} .
          docker tag final-assignment:${BUILD_NUMBER} 191351159627.dkr.ecr.us-east-1.amazonaws.com/final-assignment:${BUILD_NUMBER}
          docker push 191351159627.dkr.ecr.us-east-1.amazonaws.com/final-assignment:${BUILD_NUMBER}
          '''
        }
      }
    }

    stage('Cleanup the docker image') {
      steps {
        script {
          sh 'docker rmi 303150498045.dkr.ecr.us-east-1.amazonaws.com/final-assignment:${BUILD_NUMBER}'
          sh 'docker rmi final-assignment:${BUILD_NUMBER}'
        }
      }
    }

    stage('Deploy the application') {
      steps {
        script {
          sh '''
          aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 191351159627.dkr.ecr.us-east-1.amazonaws.com
          docker pull 191351159627.dkr.ecr.us-east-1.amazonaws.com/final-assignment:${BUILD_NUMBER}
          docker run -d -p 8080:8081 191351159627.dkr.ecr.us-east-1.amazonaws.com/final-assignment:${BUILD_NUMBER}
          '''
        }
      }
    }
  }

  options {
    buildDiscarder(logRotator(numToKeepStr: '10'))
    disableConcurrentBuilds()
    timeout(time: 1, unit: 'HOURS')
  }
}
