pipeline {
    agent any

    environment {
        APP_NAME = 'js-app'
        MONGO_NETWORK = 'mongo-network'
        PORT = '3000'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Cloning repository...'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing dependencies...'
                sh 'npm install'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh 'docker build -t ${APP_NAME} .'
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

        stage('Deploy Container') {
            steps {
                echo 'Running new container...'
                sh '''
                    docker run -d \
                        --name ${APP_NAME} \
                        --net ${MONGO_NETWORK} \
                        -p ${PORT}:${PORT} \
                        ${APP_NAME}
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
            echo 'Pipeline completed successfully! App is running on port 3000 '
        }
        failure {
            echo 'Pipeline failed! Check the logs above.'
            sh 'docker logs ${APP_NAME} || true'
        }
        always {
            echo 'Pipeline finished.'
        }
    }
}
