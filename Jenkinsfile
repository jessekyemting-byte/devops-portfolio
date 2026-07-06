pipeline {
    agent {
        docker {
            image 'node:21-alpine' 
            // Mounts your machine's Docker engine into the container so 'sh docker' works
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    stages {
        stage('Checkout') {
            steps {
                echo 'Pulling the latest Portfolio Website source code from GitHub...'
                checkout scm
            }
        }
        stage('Install Dependencies') {
            steps {
                echo 'Installing web application packages...'
                sh 'npm install' 
            }
        }
        stage('Build') {
            steps {
                echo 'Compiling static production assets...'
                sh 'npm run build --if-present'
            }
        }
        stage('Run Application') {
            steps {
                echo 'Deploying static files permanently to production Nginx server...'
                sh '''
                    docker rm -f my-live-portfolio || true
                    docker run -d -p 3000:80 --name my-live-portfolio -v "$(pwd)":/usr/share/nginx/html nginx:alpine
                '''
            }
        }
        stage('Cleanup') {
            steps {
                echo 'Cleaning up pipeline agent files...'
            }
        }
    }
}
