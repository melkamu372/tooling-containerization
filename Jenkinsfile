pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
        DOCKER_IMAGE_NAME = 'melkamu372/php-todo-app'
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout code from the repository
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def branchName = env.BRANCH_NAME
                    def imageTag = branchName == 'master' ? 'latest' : "${branchName}-latest"

                    echo "Building Docker image with tag ${imageTag}"
                    sh "docker build -t ${DOCKER_IMAGE_NAME}:${imageTag} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    def branchName = env.BRANCH_NAME
                    def imageTag = branchName == 'master' ? 'latest' : "${branchName}-latest"

                    echo "Pushing Docker image with tag ${imageTag}"
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        sh "docker push ${DOCKER_IMAGE_NAME}:${imageTag}"
                    }
                }
            }
        }
    }
}
