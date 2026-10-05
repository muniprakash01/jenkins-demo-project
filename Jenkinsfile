pipeline {
    agent any

    environment {
        IMAGE_NAME = 'jenkins-demo-project'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Hello') {
            steps {
                echo 'Starting Jenkins CI/CD Pipeline'
            }
        }

        stage('Git Checkout') {
            steps {
                echo 'Checking out source code from GitHub'

                git branch: 'main',
                    url: 'https://github.com/muniprakash01/jenkins-demo-project.git'
            }
        }

        stage('Check Python') {
            steps {
                echo 'Checking Python version'

                sh '''
                    python3 --version
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Creating Python virtual environment'

                sh '''
                    rm -rf .venv

                    python3 -m venv .venv

                    .venv/bin/python -m pip install --upgrade pip

                    .venv/bin/python -m pip install -r app/requirements.txt

                    .venv/bin/python -m pip install pytest
                '''
            }
        }

        stage('Application Test') {
            steps {
                echo 'Running application tests'

                sh '''
                    .venv/bin/python -m pytest tests/ -v
                '''
            }
        }

        stage('Docker Check') {
            steps {
                echo 'Checking Docker installation'

                sh '''
                    docker --version
                '''
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image'

                sh '''
                    docker build \
                        -t ${IMAGE_NAME}:${IMAGE_TAG} \
                        -t ${IMAGE_NAME}:latest .
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                echo 'Scanning Docker image with Trivy'

                sh '''
                    trivy image \
                        --config /dev/null \
                        --severity HIGH,CRITICAL \
                        ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Docker Compose Validation') {
            steps {
                echo 'Validating Docker Compose configuration'

                sh '''
                    docker compose config -q
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application'

                sh '''
                    export IMAGE_TAG=${IMAGE_TAG}

                    docker compose up -d
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo 'Checking application health'

                sh '''
                    echo "Waiting for application..."

                    sleep 10

                    curl -f http://localhost:5000/health

                    echo "Application is healthy"
                '''
            }
        }

        stage('Cleanup') {
            steps {
                echo 'Cleaning unused Docker images'

                sh '''
                    docker image prune -f
                '''
            }
        }
    }

    post {

        success {
            echo '========================================'
            echo 'CI/CD PIPELINE SUCCESSFUL'
            echo '========================================'
        }

        failure {
            echo '========================================'
            echo 'CI/CD PIPELINE FAILED'
            echo 'Check the Console Output'
            echo '========================================'
        }

        always {
            echo 'Jenkins pipeline execution completed'
        }
    }
}
