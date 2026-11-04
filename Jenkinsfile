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
                sh '''
                    docker run --rm -i python:3.11-slim python -c "import sys; compile(sys.stdin.read(), 'app.py', 'exec')" < app.py
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    IMAGE_TAG=$(git rev-parse --short HEAD)
                    docker build -t python-crud-api:${IMAGE_TAG} -t python-crud-api:latest .
                '''
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                    IMAGE_TAG=$(git rev-parse --short HEAD)
                    docker volume create production-api-data
                    docker rm -f production-api-container || true
                    docker run -d \
                        --name production-api-container \
                        -p 5000:5000 \
                        -v production-api-data:/var/lib/backend_data \
                        python-crud-api:${IMAGE_TAG}
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    echo "Waiting for container health check..."
                    for i in $(seq 1 15); do
                        STATUS=$(docker inspect --format='{{.State.Health.Status}}' production-api-container)
                        echo "Health status: $STATUS"

                        if [ "$STATUS" = "healthy" ]; then
                            echo "Application is healthy!"
                            exit 0
                        fi

                        if [ "$STATUS" = "unhealthy" ]; then
                            docker logs production-api-container
                            exit 1
                        fi

                        sleep 3
                    done

                    docker logs production-api-container
                    echo "Health check timeout!"
                    exit 1
                '''
            }
        }
    }
}