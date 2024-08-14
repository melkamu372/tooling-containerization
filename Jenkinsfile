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
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "${env.GIT_BRANCH}"]],
                    userRemoteConfigs: [[url: 'https://github.com/melkamu372/tooling-containerization.git']]
                ])
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
                    def imageTag = "0.0.${env.BUILD_NUMBER}"
                    def imageNameWithTag = "${DOCKER_IMAGE_NAME}:${imageTag}"

                    echo "Verifying if the image exists locally: ${imageNameWithTag}"
                    
                    def images = sh(script: "docker images -q ${imageNameWithTag}", returnStdout: true).trim()
                    
                    if (images) {
                        echo "Image found: ${imageNameWithTag}"
                    } else {
                        error "Image ${imageNameWithTag} not found. Build might have failed."
                    }
                }
            }
        }

        stage('Tag and Push Image') {
            steps {
                script {
                    def buildTag = "0.0.${env.BUILD_NUMBER}"
                    def imageNameWithTag = "${DOCKER_IMAGE_NAME}:${buildTag}"
                    def dockerHubTag = "registry.hub.docker.com/${DOCKER_IMAGE_NAME}:${buildTag}"

                    echo "Tagging image ${imageNameWithTag} as ${dockerHubTag}"
                    
                    // Tag the image
                    def tagCommand = "docker tag ${imageNameWithTag} ${dockerHubTag}"
                    if (isUnix()) {
                        sh tagCommand
                    } else {
                        bat tagCommand
                    }

                    // Push the image to Docker Hub
                    docker.withRegistry('https://registry.hub.docker.com', DOCKER_CREDENTIALS_ID) {
                        docker.image(dockerHubTag).push("${buildTag}")
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
