pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code'
                checkout scm
            }
        }

        stage('Sanity Test') {
            steps {
                echo 'Jenkins pipeline is working'
                sh 'ls -la'
            }
        }
    }
}

