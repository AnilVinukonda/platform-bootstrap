pipeline {
    agent any

    environment {
        IMAGE_NAME = "platform-bootstrap-app"
        DOCKER_REPO = "avinuko/platform-bootstrap-app"
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
                  set -eux
                  echo "Building Docker image..."
                  docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ./app
                '''
            }
        }

        stage('Health Test') {
            steps {
                sh '''
                  set -eux
                  echo "Starting test container"
                  docker rm -f test-app || true
                  docker run -d --name test-app ${IMAGE_NAME}:${IMAGE_TAG}

                  echo "Waiting for app to become healthy..."
                  for i in $(seq 1 20); do
                    IP=$(docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' test-app)
                    if curl -fsS "http://$IP:8080/health" >/dev/null; then
                      echo "Health check OK"
                      exit 0
                    fi
                    sleep 1
                  done

                  echo "Health check failed"
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

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                      set -eux
                      echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                      docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_REPO}:${IMAGE_TAG}
                      docker push ${DOCKER_REPO}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Helm Deploy') {
            steps {
                withCredentials([file(
                    credentialsId: 'kubeconfig',
                    variable: 'KUBECONFIG'
                )]) {
                    sh '''
                      set -eux
                      helm upgrade --install platform-bootstrap \
                        ./helm/platform-bootstrap-app \
                        --namespace platform-bootstrap \
                        --create-namespace \
                        --set image.repository=${DOCKER_REPO} \
                        --set image.tag=${IMAGE_TAG}
                    '''
                }
            }
        }
    }
}

