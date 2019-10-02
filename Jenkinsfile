pipeline {
  environment {
    registry = "ippeter/mysql-tester"
    registryCredential = 'dockerhub'
  }
  
  agent any
  
  stages {
    stage('Lint Python') {
      steps {
        sh 'pylint --disable=R,C,W1203 mysql-tester.py'
      }
    }

    stage('Lint HTML') {
      steps {
        sh 'tidy -q -e hello.html'
      }
    }

    stage('Lint Dockerfile') {
      steps {
        sh 'hadolint --ignore DL3013 Dockerfile'
      }
    }

    stage('Build Image') {
      steps {
        script {
          dockerImage = docker.build registry + ":$BUILD_NUMBER"
        }
      }
    }
    
    stage('Push Image') {
      steps {
        script {
          docker.withRegistry( '', registryCredential ) {
            dockerImage.push()
          }
        }
      }
    }

    stage('Check Minikube') {
      steps {
        script {
          withKubeConfig([credentialsId: 'JenkinsToken', serverUrl: 'https://192.168.0.215:8443']) {
            sh 'kubectl cluster-info'
          }
        }
      }
    }

    stage('Deploy Image') {
      steps {
        sh '''
          if kubectl get deployment | grep -q mysql-tester
          then
            echo "Deployment found, updating..."
          else
            kubectl create deployment mysql-tester --image=$dockerImage
            kubectl scale deployment mysql-tester --replicas=2
          fi
        '''
      }
    }

  }
}
