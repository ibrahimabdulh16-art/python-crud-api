pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate Python') {
            steps {
                sh 'docker run --rm -v "$WORKSPACE:/app" -w /app python:3.11-slim python -m py_compile app.py'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t python-crud-api:latest .'
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                    docker volume create production-api-data
                    docker rm -f production-api-container || true
                    docker run -d \
                        --name production-api-container \
                        -p 5000:5000 \
                        -v production-api-data:/var/lib/backend_data \
                        python-crud-api:latest
                '''
            }
        }
    }
}