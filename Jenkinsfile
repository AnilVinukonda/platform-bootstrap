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
                  docker version
                  docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ./app
                  docker images | grep ${IMAGE_NAME}
                '''
            }
        }
    }
}

