pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
        DOCKER_IMAGE_NAME = 'melkamu372/php-todo-app'
        COMPOSE_FILE = "${WORKSPACE}/tooling.yml" // Path to Docker Compose file
    }

    stages {
        stage('Initial Cleanup') {
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

                    // Verify if the image exists
                    def buildTag = "0.0.${env.BUILD_NUMBER}"
                    def imageNameWithTag = "${DOCKER_IMAGE_NAME}:${buildTag}"
                    def imagesList = isUnix()
                        ? sh(script: "docker images", returnStdout: true).trim()
                        : bat(script: "docker images", returnStdout: true).trim()

                    echo "Docker Images:\n${imagesList}"

                    // Verify if the image exists with the tag
                    def imageId = isUnix()
                        ? sh(script: "docker images -q ${imageNameWithTag}", returnStdout: true).trim()
                        : bat(script: "docker images -q ${imageNameWithTag}", returnStdout: true).trim()

                    if (imageId) {
                        echo "Image ${imageNameWithTag} is present."
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

                    echo "Tagging image ${DOCKER_IMAGE_NAME}:latest as ${imageNameWithTag}"

                    def tagCommand = "docker tag ${DOCKER_IMAGE_NAME}:latest ${imageNameWithTag}"
                    if (isUnix()) {
                        sh tagCommand
                    } else {
                        bat tagCommand
                    }

                    echo "Pushing image to Docker Hub"
                    docker.withRegistry('https://registry.hub.docker.com', DOCKER_CREDENTIALS_ID) {
                        docker.image(imageNameWithTag).push("${buildTag}")
                    }
                }
            }
        }

        stage('Cleanup') {
            steps {
                script {
                    echo "Cleaning up Docker Compose setup and image"

                    def cleanupCommand = "docker-compose -f \"${COMPOSE_FILE}\" down"
                    if (isUnix()) {
                        sh cleanupCommand
                    } else {
                        bat cleanupCommand
                    }

                    // Only attempt to remove the image if it exists
                    def removeImageCommand = "docker rmi ${DOCKER_IMAGE_NAME}:0.0.${env.BUILD_NUMBER}"
                    if (isUnix()) {
                        sh removeImageCommand
                    } else {
                        bat removeImageCommand
                    }
                }
            }
        }
    }
}
