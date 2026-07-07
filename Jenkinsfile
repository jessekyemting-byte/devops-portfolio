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
                // Stops and removes the container if it exists; || true prevents crashing if it doesn't exist yet
                sh 'docker stop kyemting-site || true'
                sh 'docker rm kyemting-site || true'
                
                echo 'Building the fresh Docker image...'
                sh 'docker build -t website-img .'
                
                echo 'Launching the new application container...'
                sh 'docker run -d -p 80:80 --name kyemting-site website-img'
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
