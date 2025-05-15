pipeline {
    agent any 

    environment {
        DOCKERHUB_CREDENTIALS = credentials('DOCKERHUB_CREDENTIALS')
        BACKEND_IMAGE = 'aungkowin/my-backend:latest'
        FRONTEND_IMAGE = 'aungkowin/my-frontend:latest'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/aungkowinitlay/Jenkins.-Upgrade-of-Lab1.git'
            }
        }

        stage('Build Backend Image') {
            steps {
                sh """
                    docker build -t ${BACKEND_IMAGE} -f backend/Dockerfile ./backend
                """
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh """
                    docker build -t ${FRONTEND_IMAGE} -f frontend/Dockerfile ./frontend
                """
            }
        }

        stage('Test Backend') {
            steps {
                sh """
                    docker run --rm -e PYTHONPATH=/app ${BACKEND_IMAGE} pytest
                """
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'DOCKERHUB_CREDENTIALS', usernameVariable: 'DOCKERHUB_CREDENTIALS_USR', passwordVariable: 'DOCKERHUB_CREDENTIALS_PSW')]) {
                    sh """
                        echo "${DOCKERHUB_CREDENTIALS_PSW}" | docker login -u "${DOCKERHUB_CREDENTIALS_USR}" --password-stdin
                    """
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                sh """
                    docker push ${BACKEND_IMAGE}
                    docker push ${FRONTEND_IMAGE}
                """
            }
        }

        stage('Deploy') {
            steps {
                sh """
                    docker rm -f backend-container || true
                    docker rm -f frontend-container || true
                    docker run -d --name backend-container -p 5000:5000 ${BACKEND_IMAGE}
                    docker run -d --name frontend-container -p 80:80 ${FRONTEND_IMAGE}
                """
            }
        }
    }

    post {
        always {
            sh """
                docker logout
                docker image prune -f
            """
        }
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}