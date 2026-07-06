pipeline {
    agent {
        docker {
            image 'node:21-alpine' 
            args '-p 3000:3000'    
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
                echo 'Launching the Portfolio live...'
                sh 'npm start &' 
            }
        }
        stage('Cleanup') {
            steps {
                echo 'Cleaning up workspace to free up server storage...'
                cleanWs()
            }
        }
    }
}
