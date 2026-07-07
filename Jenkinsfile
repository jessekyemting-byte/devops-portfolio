pipeline {
    agent {
        label 'production'
    }

    options {
        timeout(time: 1, unit: 'HOURS')
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
                sh 'docker --version'
            }
        }

        stage('Build & Deploy') {
            steps {
                echo 'Stopping and cleaning up any older container instances...'
                sh 'docker compose down --remove-orphans || true'
                
                echo 'Building images and launching the application containers...'
                sh 'docker compose up -d --build'
            }
        }
    }

    post {
        success {
            echo 'Pipeline successfully executed and app deployed on the remote AWS server!'
        }
        failure {
            echo 'Something went wrong during execution.'
        }
    }
}
