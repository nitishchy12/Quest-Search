pipeline {
    agent any

    environment {
        FRONTEND_IMAGE = 'quest-search-frontend:latest'
        BACKEND_IMAGE  = 'quest-search-backend:latest'
    }

    stages {
        stage('Clone Repository') {
            steps {
                echo 'Cloning the repository...'
                checkout scm
            }
        }

        stage('Build Docker Images') {
            steps {
                echo 'Building Docker images for frontend and backend...'
                script {
                    bat "docker build -t ${env.FRONTEND_IMAGE} ./apps/frontend"
                    bat "docker build -t ${env.BACKEND_IMAGE} ./apps/backend"
                }
            }
        }

        stage('Run Containers') {
            steps {
                echo 'Running frontend and backend containers...'
                script {
                    bat "docker run -d --name frontend -p 3000:3000 ${env.FRONTEND_IMAGE}"
                    bat "docker run -d --name backend -p 8080:8080 ${env.BACKEND_IMAGE}"
                }
            }
        }

        stage('Push Images to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    bat '''
                        echo %DOCKER_PASS% | docker login --username %DOCKER_USER% --password-stdin
                        
                        docker tag quest-search-frontend:latest %DOCKER_USER%/quest-search-frontend:latest
                        docker tag quest-search-backend:latest %DOCKER_USER%/quest-search-backend:latest
                        docker push %DOCKER_USER%/quest-search-frontend:latest
                        docker push %DOCKER_USER%/quest-search-backend:latest
                    '''
                }
            }
        }

        stage('Done') {
            steps {
                echo 'Quest Search: Build, Run, and Push complete!'
            }
        }
    }

    post {
        always {
            echo 'Cleaning up unused Docker resources...'
            bat 'docker system prune -f'
        }
    }
}
