pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME = "flask-cicd"
        CONTAINER_NAME = "flask-cicd-app"
        APP_PORT = "5000"
        APP_ENV = "production"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies and Test') {
    steps {
        sh '''
            set -eu

            python3 -m venv .venv

            .venv/bin/python -m ensurepip --upgrade

            .venv/bin/python -m pip install --upgrade pip

            .venv/bin/python -m pip install -r app/requirements.txt pytest

            .venv/bin/python -m pytest -v
        '''
    }
}

        stage('Build Docker Image') {
            steps {
                script {
                    env.IMAGE_TAG = "${BUILD_NUMBER}"
                }

                sh '''
                    set -eu

                    docker build \
                        --pull \
                        -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Trivy Security Scan') {
            steps {
                sh '''
                    set -eu

                    trivy image \
                        --exit-code 1 \
                        --severity HIGH,CRITICAL \
                        --ignore-unfixed \
                        ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Deploy') {
            steps {
                script {
                    env.PREVIOUS_TAG = sh(
                        script: '''
                            docker inspect \
                                --format='{{.Config.Image}}' \
                                ${CONTAINER_NAME} 2>/dev/null \
                            | sed "s|^${IMAGE_NAME}:||" || true
                        ''',
                        returnStdout: true
                    ).trim()

                    if (env.PREVIOUS_TAG == env.IMAGE_TAG) {
                        env.PREVIOUS_TAG = ""
                    }
                }

                sh '''
                    set -eu

                    export IMAGE_NAME=${IMAGE_NAME}
                    export IMAGE_TAG=${IMAGE_TAG}
                    export APP_ENV=${APP_ENV}

                    docker compose up -d --no-build
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    set -eu

                    for i in $(seq 1 30); do
                        STATUS=$(docker inspect \
                            --format='{{.State.Health.Status}}' \
                            ${CONTAINER_NAME} 2>/dev/null || true)

                        if [ "$STATUS" = "healthy" ]; then
                            echo "Application is healthy"
                            exit 0
                        fi

                        echo "Waiting for application: $STATUS"
                        sleep 2
                    done

                    echo "Health check failed"
                    exit 1
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully.'

            sh '''
                set -eu

                docker image prune -f
            '''
        }

        failure {
            echo 'Pipeline failed. Checking whether rollback is possible.'

            script {
                if (env.PREVIOUS_TAG?.trim()) {
                    sh '''
                        set -eu

                        echo "Rolling back to ${PREVIOUS_TAG}"

                        export IMAGE_NAME=${IMAGE_NAME}
                        export IMAGE_TAG=${PREVIOUS_TAG}
                        export APP_ENV=${APP_ENV}

                        docker compose up -d --no-build

                        for i in $(seq 1 30); do
                            STATUS=$(docker inspect \
                                --format='{{.State.Health.Status}}' \
                                ${CONTAINER_NAME} 2>/dev/null || true)

                            if [ "$STATUS" = "healthy" ]; then
                                echo "Rollback successful"
                                exit 0
                            fi

                            sleep 2
                        done

                        echo "Rollback health check failed"
                        exit 1
                    '''
                } else {
                    echo 'No previous deployment found. Manual recovery may be required.'
                }
            }
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}
