pipeline {
    agent any

    environment {
        IMAGE_NAME = 'jenkins-demo-project'
    }

    stages {

        stage('Hello') {
            steps {
                echo 'Starting Jenkins CI/CD Pipeline'
            }
        }

        stage('Python Version') {
            steps {
                sh 'python3 --version'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    rm -rf .venv
                    python3 -m venv .venv
                    .venv/bin/python -m pip install -r app/requirements.txt
                    .venv/bin/python -m pip install pytest
                '''
            }
        }

        stage('Application Test') {
            steps {
                sh '''
                    .venv/bin/python -m pytest tests/ -v
                '''
            }
        }

        stage('Docker Check') {
            steps {
                sh 'docker --version'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t ${IMAGE_NAME}:test .
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                    trivy image \
                    --config /dev/null \
                    --severity HIGH,CRITICAL \
                    ${IMAGE_NAME}:test
                '''
            }
        }

        stage('Docker Compose Validation') {
            steps {
                sh 'docker compose config -q'
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                    docker compose up -d
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    echo "Waiting for application to start..."
                    sleep 10

                    echo "Checking application health..."

                    curl -f http://localhost:5000/health

                    echo "Application health check passed successfully."
                '''
            }
        }

        stage('Pipeline Completed') {
            steps {
                echo 'All CI/CD testing and health check stages completed successfully'
            }
        }
    }

    post {
        success {
            echo 'CI/CD PIPELINE SUCCESSFUL'
        }

        failure {
            echo 'CI/CD PIPELINE FAILED'
        }

        always {
            echo 'Jenkins pipeline execution completed'
        }
    }
}
