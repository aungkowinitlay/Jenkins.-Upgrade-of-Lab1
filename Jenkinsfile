pipeline {
    agent any 
    environment {
        DOCKERHUB_CRENDENTIALS = crendentials ('DOCKERHUB_CRENDENTIALS')
        BACKEND-IMAGE = aungkowin/my-backend:latest
        FRONTEND-IMAGE = aungkowin/my-frontend:latest
    }
    stages {
        stage('Checkout'){
            steps {
                //Checkout code from GitHub
                
                    git branch: 'main', url: 'https://github.com/aungkowinitlay/Jenkins.-Upgrade-of-Lab1.git'
                
            }
        }
        stage('Build Backend Image'){
            steps{
                //Build Backend Docker Image
                sh """
                docker build -t ${BACKEND-IMAGE} -f backend/Dockerfile ./backend
                """
            }
        }
        stage('Build Frontend Image'){
            steps{
                //Buil Frontend Docker image
                sh """
                docker build -t ${FRONTEND-IMAGE} -t frontend/Dockerfile ./frontend
                """
            }
        }
        stage('Test Backend'){
            steps{
                sh """
                docker run --rm  ${BACKEND-IMAGE} pytest
                """

            }
        }
        stage('Docker Login'){
            steps{
                //Docker Hub Login
                echo $DOCKERHUB_CRENDENTIALS_PSW | docker Login -u $DOCKERHUB_CRENDENTIALS_USR --password-stdin
            }
        }
        stages('Push Docker Image'){
            steps{
                sh """
                docker push ${BACKEND-IMAGE}
                docker push ${FRONTEND-IMAGE}
                """
            }
        }
        stage('Deploy'){
            steps{
                sh """
                docker rm -f backend-container || true
                docker rm -f frontend-container || true
                docker run -d --name backend-container -p 5000:5000 ${BACKEND-IMAGE}
                docker run -d --name frontend-container -p 80:80 ${FRONTEND-IMAGE}
                """
            }
        }

    }
}
post {
    always {
        node (label: ''){
            sh """
                docker logout
                docker image prune -f
                """
        }
    }
    success {
        node (label: ''){
            echo "Pipeline completed successfully!"
        }
    }
    failure {
        node (label: ''){
            echo "Pipeline failed!"
        }
    }
}