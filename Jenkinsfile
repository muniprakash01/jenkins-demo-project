```groovy
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        APP_NAME = 'jenkins-demo-project'
        IMAGE_NAME = 'jenkins-demo-project'
        CONTAINER_PORT = '5000'
        HOST_PORT = '5000'
        COMPOSE_FILE = 'docker-compose.yml'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo 'Source code checked out successfully.'
            }
        }

        stage('Install Dependencies and Test') {
            steps {
                sh '''
                    set -eu

                    sudo apt-get update
                    sudo apt-get install -y python3-venv python3-pip

                    rm -rf .venv
                    python3 -m venv .venv

                    .venv/bin/python -m pip install --upgrade pip
                    .venv/bin/python -m pip install -r app/requirements.txt pytest

                    .venv/bin/python -m pytest -v
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    set -eu
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                    docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Trivy Security Scan') {
            steps {
                sh '''
                    set -eu
                    trivy image --exit-code 1 \
                        --severity HIGH,CRITICAL \
                        ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    set -eu

                    docker compose -f ${COMPOSE_FILE} \
                        up -d --build
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    set -eu

                    for i in $(seq 1 30); do
                        if curl -fsS http://localhost:${HOST_PORT}/; then
                            echo "Application is healthy."
                            exit 0
                        fi

                        echo "Waiting for application..."
                        sleep 5
                    done

                    echo "Health check failed."
                    exit 1
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check the console output.'

            sh '''
                docker compose -f ${COMPOSE_FILE} ps || true
            '''
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}
```
