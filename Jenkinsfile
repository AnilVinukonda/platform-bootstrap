pipeline {
  agent any

  environment {
    KUBECONFIG = "/var/jenkins_home/.kube/config"
    IMAGE_NAME = "avinuko/platform-bootstrap-app"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Docker Image') {
      steps {
        sh '''
          set -eux
          docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
        '''
      }
    }

    stage('Push Image') {
      steps {
        sh '''
          set -eux
          docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest
          docker push ${IMAGE_NAME}:${BUILD_NUMBER}
          docker push ${IMAGE_NAME}:latest
        '''
      }
    }

    stage('Deploy to Kubernetes (Helm)') {
      steps {
        sh '''
          set -eux
          helm upgrade --install platform-bootstrap-app ./helm/platform-bootstrap-app \
            --namespace default \
            --create-namespace \
            --set image.repository=${IMAGE_NAME} \
            --set image.tag=${BUILD_NUMBER}

          kubectl rollout status deployment/platform-bootstrap-app --timeout=120s
        '''
      }
    }
  }

  post {
    success {
      echo "Deployment successful"
    }
    failure {
      echo "Deployment failed"
    }
  }
}

