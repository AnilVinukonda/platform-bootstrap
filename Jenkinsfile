

pipeline {
  agent any

  environment {
    IMAGE_NAME   = "avinuko/platform-bootstrap-app"
    KUBECONFIG   = "/var/jenkins_home/.kube/config"
    HELM_RELEASE = "platform-bootstrap-app"
    K8S_NS       = "default"
    APP_PORT     = "8080"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
        sh '''
          set -eux
          echo "Workspace: $(pwd)"
          ls -la
        '''
      }
    }

    stage('Verify Tools') {
      steps {
        sh '''
          set -eux
          docker version
          kubectl version --client
          helm version
          kubectl --kubeconfig "$KUBECONFIG" get nodes
        '''
      }
    }

    stage('Build Docker Image') {
      steps {
        sh '''
          set -eux
          test -f ./app/Dockerfile
          docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ./app
        '''
      }
    }

    stage('Smoke Test (Local Container)') {
      steps {
        sh '''
          set -eux

          # Run container locally on Jenkins host network namespace (inside the VM)
          docker rm -f pb-smoke || true
          docker run -d --name pb-smoke -p 18080:${APP_PORT} ${IMAGE_NAME}:${BUILD_NUMBER}

          # Wait and test /health
          for i in $(seq 1 30); do
            if curl -fsS http://localhost:18080/health >/dev/null; then
              echo "Health check passed"
              break
            fi
            echo "Waiting for /health... attempt $i"
            sleep 2
          done

          curl -fsS http://localhost:18080/health
          docker rm -f pb-smoke || true
        '''
      }
    }

    stage('Push Docker Image (DockerHub)') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-creds',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh '''
            set -eux
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

            docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest
            docker push ${IMAGE_NAME}:${BUILD_NUMBER}
            docker push ${IMAGE_NAME}:latest
          '''
        }
      }
    }

    stage('Deploy to Kubernetes (Helm)') {
      steps {
        sh '''
          set -eux

          helm upgrade --install ${HELM_RELEASE} ./helm/platform-bootstrap-app \
            --namespace ${K8S_NS} \
            --create-namespace \
            --set image.repository=${IMAGE_NAME} \
            --set image.tag=${BUILD_NUMBER}

          kubectl --kubeconfig "$KUBECONFIG" -n ${K8S_NS} rollout status deployment/${HELM_RELEASE} --timeout=180s
        '''
      }
    }

    stage('Health Check (Kubernetes)') {
      steps {
        sh '''
          set -eux

          # Show useful debug info
          kubectl --kubeconfig "$KUBECONFIG" -n ${K8S_NS} get deploy,po,svc -l app.kubernetes.io/instance=${HELM_RELEASE} || true
          kubectl --kubeconfig "$KUBECONFIG" -n ${K8S_NS} get endpoints ${HELM_RELEASE} || true

          # Port-forward service and curl /health
          kubectl --kubeconfig "$KUBECONFIG" -n ${K8S_NS} port-forward svc/${HELM_RELEASE} 18081:${APP_PORT} >/tmp/pf.log 2>&1 &
          PF_PID=$!

          # Wait for port-forward to be ready
          for i in $(seq 1 30); do
            if curl -fsS http://localhost:18081/health >/dev/null; then
              echo "Kubernetes /health passed"
              break
            fi
            echo "Waiting for Kubernetes /health... attempt $i"
            sleep 2
          done

          curl -fsS http://localhost:18081/health

          # Cleanup port-forward
          kill $PF_PID || true
        '''
      }
    }
  }

  post {
    always {
      sh '''
        echo "==== Final Cluster Snapshot ===="
        kubectl --kubeconfig "$KUBECONFIG" -n default get deploy,po,svc || true
      '''
    }
    success {
      echo "Pipeline completed successfully: Build → Test → Push → Deploy → Health OK"
    }
    failure {
      echo "Pipeline failed. Check stage logs above."
    }
  }
}

