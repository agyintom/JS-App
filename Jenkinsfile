pipeline {
    agent any

    environment {
        APP_NAME = 'js-app'
        MONGO_NETWORK = 'mongo-network'
        PORT = '3000'
        IMAGE_TAG = "${APP_NAME}:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Cloning branch: ${env.BRANCH_NAME}"
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image: ${IMAGE_TAG}"
                sh 'docker build -t ${IMAGE_TAG} .'
            }
        }

        stage('Stop Old Container') {
            steps {
                echo 'Stopping old container if running...'
                sh '''
                    docker stop ${APP_NAME} || true
                    docker rm ${APP_NAME} || true
                '''
            }
        }

        stage('Deploy') {
            when {
                anyOf {
                    branch 'main'
                    branch 'master'
                    branch 'dev'
                }
            }
            steps {
                echo "Deploying branch: ${env.BRANCH_NAME}"
                sh '''
                    docker run -d \
                        --name ${APP_NAME} \
                        --net ${MONGO_NETWORK} \
                        -p ${PORT}:${PORT} \
                        ${IMAGE_TAG}
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo 'Checking app is running...'
                sh 'sleep 5 && docker logs ${APP_NAME}'
            }
        }

    }

    post {
        success {
            echo "✅ Pipeline succeeded on branch: ${env.BRANCH_NAME}"
        }
        failure {
            echo "❌ Pipeline failed on branch: ${env.BRANCH_NAME}"
            sh 'docker logs ${APP_NAME} || true'
        }
        always {
            echo "Pipeline finished - Branch: ${env.BRANCH_NAME} Build: ${env.BUILD_NUMBER}"
        }
    }
}