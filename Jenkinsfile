pipeline {
    agent any // Runs on the host machine where the docker CLI is already fully installed
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Pulling the latest Portfolio Website source code from GitHub...'
                checkout scm
            }
        }
        stage('Build Assets') {
            steps {
                echo 'Building static assets using a temporary Node container...'
                // This runs node dynamically on the host, installs dependencies, and builds
                sh 'docker run --rm -v "$(pwd)":/app -w /app node:21-alpine sh -c "npm install && npm run build --if-present"'
            }
        }
        stage('Run Application') {
            steps {
                echo 'Deploying static files permanently to production Nginx server...'
                // Now this executes seamlessly on the host shell where docker is native
                sh '''
                    docker rm -f my-live-portfolio || true
                    docker run -d -p 3000:80 --name my-live-portfolio -v "$(pwd)":/usr/share/nginx/html nginx:alpine
                '''
            }
        }
    }
}
