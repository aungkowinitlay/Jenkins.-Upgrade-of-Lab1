pipeline {
    agent { label 'docker-agent' } // Specify agent with Docker installed

    environment {
        DOCKERHUB_CREDENTIALS = credentials('DOCKERHUB_CREDENTIALS')
        BACKEND_IMAGE = 'aungkowin/my-backend:latest'
        FRONTEND_IMAGE = 'aungkowin/my-frontend:latest'
    }

    stages {
        stage('Checkout') {
            steps {
                retry(3) {
                    git branch: 'main', url: 'https://github.com/aungkowinitlay/Jenkins.-Upgrade-of-Lab1.git'
                }
            }
        }

        stage('Build Images') {
            parallel {
                stage('Build Backend') {
                    steps {
                        timeout(time: 10, unit: 'MINUTES') {
                            sh "docker build -t ${BACKEND_IMAGE} -f backend/Dockerfile ./backend"
                        }
                    }
                }
                stage('Build Frontend') {
                    steps {
                        timeout(time: 10, unit: 'MINUTES') {
                            sh "docker build -t ${FRONTEND_IMAGE} -f frontend/Dockerfile ./frontend"
                        }
                    }
                }
            }
        }

        stage('Test Backend') {
            steps {
                script {
                    try {
                        sh "docker run --rm ${BACKEND_IMAGE} pytest"
                    } catch (Exception e) {
                        echo "Backend tests failed: ${e}"
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }

        stage('Docker Login') {
            steps {
                retry(3) {
                    sh "echo '${DOCKERHUB_CREDENTIALS_PSW}' | docker login -u '${DOCKERHUB_CREDENTIALS_USR}' --password-stdin"
                }
            }
        }

        stage('Push Images') {
            parallel {
                stage('Push Backend') {
                    steps {
                        sh "docker push ${BACKEND_IMAGE}"
                    }
                }
                stage('Push Frontend') {
                    steps {
                        sh "docker push ${FRONTEND_IMAGE}"
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sh """
                    docker rm -f backend-container || echo 'Failed to remove backend-container'
                    docker rm -f frontend-container || echo 'Failed to remove frontend-container'
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
                docker container prune -f
            """
        }
        success {
            slackSend(channel: '#ci', message: "Pipeline completed successfully for ${env.JOB_NAME} #${env.BUILD_NUMBER}")
        }
        failure {
            slackSend(channel: '#ci', message: "Pipeline failed for ${env.JOB_NAME} #${env.BUILD_NUMBER}")
        }
    }
}