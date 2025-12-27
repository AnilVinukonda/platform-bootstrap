pipeline {
    agent any

    environment {
        IMAGE_NAME = "platform-bootstrap-app"
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                  echo "Building Docker image..."
                  docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ./app
                '''
            }
        }

        stage('Health Test') {
            steps {
                sh '''
                  echo "Running container for health test"
                  docker rm -f test-app || true

                  docker run -d --name test-app -p 18080:8080 ${IMAGE_NAME}:${IMAGE_TAG}

                  sleep 5

                  echo "Calling /health"
                  curl -f http://localhost:18080/health

                  echo "Health check passed"
                  docker rm -f test-app
                '''
            }
        }
    }
}

