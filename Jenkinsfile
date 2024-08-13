pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
        DOCKER_IMAGE_NAME = 'melkamu372/php-todo-app'
        COMPOSE_FILE = "${WORKSPACE}\\tooling.yml"  // Specify the absolute path to the Docker Compose file
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

        stage('Verify Files') {
            steps {
                script {
                    echo "Current directory: ${pwd()}"
                    echo "Listing files in workspace:"
                    bat "dir ${WORKSPACE}"
                    echo "Docker Compose file path: ${COMPOSE_FILE}"
                }
            }
        }

        stage('Build and Start Containers') {
            steps {
                script {
                    // Pull any required images and start the containers
                    def buildCommand = "docker-compose -f \"${COMPOSE_FILE}\" up -d --build"
                    
                    echo "Executing: ${buildCommand}"
                    
                    if (isUnix()) {
                        sh buildCommand
                    } else {
                        bat buildCommand
                    }
                }
            }
        }

        stage('Smoke Test') {
            steps {
                script {
                    def httpEndpoint = "http://localhost:5000"  // Change to the actual endpoint of your service
                    def responseCode = sh(script: "curl -o /dev/null -s -w '%{http_code}' ${httpEndpoint}", returnStdout: true).trim()
                    if (responseCode != '200') {
                        error "Expected status code 200 but got ${responseCode}"
                    }
                }
            }
        }

        stage('Push Image') {
            steps {
                script {
                    def buildTag = "0.0.${env.BUILD_NUMBER}"
                    def imageNameWithTag = "${DOCKER_IMAGE_NAME}:${buildTag}"
                    
                    docker.withRegistry('https://registry.hub.docker.com', DOCKER_CREDENTIALS_ID) {
                        docker.image(imageNameWithTag).push("${buildTag}")
                    }
                }
            }
        }

        stage('Cleanup') {
            steps {
                cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenUnstable: true, deleteDirs: true)
                
                script {
                    def cleanupCommand = "docker-compose -f \"${COMPOSE_FILE}\" down && docker rmi ${DOCKER_IMAGE_NAME}:0.0.${env.BUILD_NUMBER} || true"
                    
                    if (isUnix()) {
                        sh cleanupCommand
                    } else {
                        bat cleanupCommand
                    }
                }
            }
        }
    }
}
