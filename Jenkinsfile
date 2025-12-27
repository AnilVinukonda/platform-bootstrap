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
                  set -eux

                  docker rm -f test-app || true
                  docker run -d --name test-app ${IMAGE_NAME}:${IMAGE_TAG}

                  for i in $(seq 1 20); do
                    IP=$(docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' test-app)
                    if curl -fsS "http://$IP:8080/health" >/dev/null; then
                      echo "Health check OK"
                      exit 0
                    fi
                    sleep 1
                  done

                  docker logs test-app || true
                  exit 1
                '''
            }
            post {
                always {
                    sh 'docker rm -f test-app || true'
                }
            }
        }
    }
}

