pipeline {
    agent {
        label 'production'
    }

    options {
        timeout(time: 1, unit: 'HOURS')
        ansiColor('xterm')
    }

    stages {
        stage('Checkout Source') {
            steps {
                echo 'Pulling the latest code repository onto the AWS agent...'
                checkout scm
            }
        }

        stage('Environment Check') {
            steps {
                echo 'Verifying dependencies on the remote Ubuntu server...'
                sh 'java -version'
                sh 'docker --version || echo "Docker not installed yet"'
            }
        }

        stage('Build & Deploy') {
            steps {
                echo 'Your custom application build or deployment commands run here.'
                // Example: sh 'docker compose up -d --build'
            }
        }
    }

    post {
        success {
            echo 'Pipeline successfully executed on the remote AWS server!'
        }
        failure {
            echo 'Something went wrong during execution.'
        }
    }
}
