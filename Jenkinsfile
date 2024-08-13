pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
        DOCKER_IMAGE_NAME = 'melkamu372/php-todo-app'
    }

    stages {
        stage("Initial cleanup") {
            steps {
                dir("${WORKSPACE}") {
                    deleteDir()
                }
            }
        }

        stage('SCM Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/melkamu372/tooling-containerization.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def branchName = env.BRANCH_NAME ?: 'main'
                    def sanitizedBranchName = branchName.replaceAll('/', '-').toLowerCase()
                    def buildTag = "${sanitizedBranchName}-0.0.${env.BUILD_NUMBER}"
                    def buildCommand = "docker build -t ${DOCKER_IMAGE_NAME}:${buildTag} ."
                    
                    bat buildCommand
                }
            }
        }

        stage('Push Image') {
            steps {
                script {
                    def branchName = env.BRANCH_NAME ?: 'main'
                    def sanitizedBranchName = branchName.replaceAll('/', '-').toLowerCase()
                    def buildTag = "${sanitizedBranchName}-0.0.${env.BUILD_NUMBER}"
                    def imageNameWithTag = "${DOCKER_IMAGE_NAME}:${buildTag}"
                    
                    docker.withRegistry('https://registry.hub.docker.com', DOCKER_CREDENTIALS_ID) {
                        docker.image(imageNameWithTag).push("${env.BUILD_NUMBER}")
                    }
                }
            }
        }

        stage('Cleanup') {
            steps {
                cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenUnstable: true, deleteDirs: true)
            }
        }
    }
}
