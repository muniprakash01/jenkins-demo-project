pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        timestamps()
        disableConcurrentBuilds()
        timeout(time: 30, unit: 'MINUTES')
    }

    environment {
        APP_NAME    = 'jenkins-demo-project'
        IMAGE_NAME  = 'jenkins-demo-project'
        COMPOSE_FILE = 'docker-compose.yml'
        APP_PORT    = '5000'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo 'GitHub checkout successful'
            }
        }

        stage('Validate Project Files') {
            steps {
                sh '''
                    set -eu
                    test -f Dockerfile
                    test -f docker-compose.yml
                    test -f app/requirements.txt
                    echo "Required project files found."
                '''
            }
        }

        stage('Install Dependencies and Test') {
            steps {
                sh '''
                    set -eu

                    python3 --version
                    python3 -m venv .venv

                    .venv/bin/python -m pip install --upgrade pip
                    .venv/bin/python -m pip install -r app/requirements.txt

                    if [ -d tests ]; then
                        .venv/bin/python -m pip install pytest
                        .venv/bin/python -m pytest -v tests
                    else
                        .venv/bin/python -m pip install pytest
                        .venv/bin/python -m pytest -v
                    fi
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    set -eu

                    docker build \
                        -t ${IMAGE_NAME}:${BUILD_NUMBER} \
                        -t ${IMAGE_NAME}:latest .
                '''
            }
        }

        stage('Trivy Security Scan') {
            steps {
                sh '''
                    set -eu

                    trivy image \
                        --severity HIGH,CRITICAL \
                        --exit-code 1 \
                        ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    set -eu

                    export IMAGE_TAG=${BUILD_NUMBER}

                    docker compose \
                        -f ${COMPOSE_FILE} \
                        up -d
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    set -eu

                    echo "Waiting for application to become healthy..."

                    for i in $(seq 1 30); do
                        if curl --fail --silent \
                            http://localhost:${APP_PORT}/ > /dev/null; then
                            echo "Application is healthy."
                            exit 0
                        fi

                        echo "Attempt $i/30: application not ready yet."
                        sleep 5
                    done

                    echo "Health check failed."
                    exit 1
                '''
            }
        }

        stage('Image Cleanup') {
            steps {
                sh '''
                    set -eu

                    docker image prune -f
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check the stage logs.'
        }

        aborted {
            echo 'Pipeline was aborted.'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}
