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
                    if (isUnix()) {
                        sh "ls -la ${WORKSPACE}"
                    } else {
                        bat "dir ${WORKSPACE}"
                    }
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

                    def imagesList
                    if (isUnix()) {
                        imagesList = sh(script: "docker images", returnStdout: true).trim()
                    } else {
                        imagesList = bat(script: "docker images", returnStdout: true).trim()
                    }
                    echo "Docker Images:\n${imagesList}"

                    def latestTag = "${DOCKER_IMAGE_NAME}:latest"
                    def buildTag = "0.0.${env.BUILD_NUMBER}"
                    def imageNameWithTag = "${DOCKER_IMAGE_NAME}:${buildTag}"

                    def latestImageId
                    if (isUnix()) {
                        latestImageId = sh(script: "docker images -q ${latestTag}", returnStdout: true).trim()
                    } else {
                        latestImageId = bat(script: "docker images -q ${latestTag}", returnStdout: true).trim()
                    }
                    echo "Checking for latest image ID: ${latestImageId}"

                    if (!latestImageId) {
                        error "Image ${latestTag} not found. Build might have failed."
                    }

                    def tagCommand = "docker tag ${latestTag} ${imageNameWithTag}"
                    echo "Tagging image ${latestTag} as ${imageNameWithTag}"
                    if (isUnix()) {
                        sh tagCommand
                    } else {
                        bat tagCommand
                    }

                    // Check for the versioned image ID
                    def imageId
                    if (isUnix()) {
                        imageId = sh(script: "docker images -q ${imageNameWithTag}", returnStdout: true).trim()
                    } else {
                        imageId = bat(script: "docker images -q ${imageNameWithTag}", returnStdout: true).trim()
                    }
                    echo "Checking for versioned image ID: ${imageId}"

                    if (!imageId) {
                        error "Image ${imageNameWithTag} not found. Build might have failed."
                    } else {
                        echo "Image ${imageNameWithTag} is present."
                    }
                }
            }
        }

        stage('Smoke Test') {
            steps {
                script {
                    def httpEndpoint = "http://localhost:5000" 
                    def responseCode = isUnix()
                        ? sh(script: "curl -o /dev/null -s -w '%{http_code}' ${httpEndpoint}", returnStdout: true).trim()
                        : bat(script: "curl -o nul -s -w %%{http_code} ${httpEndpoint}", returnStdout: true).trim()

                    echo "Response code: ${responseCode}"

                    if (responseCode != '200') {
                        error "Expected status code 200 but got ${responseCode}"
                    }
                }
            }
        }

        stage('Tag and Push Image') {
            steps {
                script {
                    def buildTag = "0.0.${env.BUILD_NUMBER}"
                    def imageNameWithTag = "${DOCKER_IMAGE_NAME}:${buildTag}"
                    def latestTag = "${DOCKER_IMAGE_NAME}:latest"

                    echo "Tagging image ${latestTag} as ${imageNameWithTag}"

                    def tagCommand = "docker tag ${latestTag} ${imageNameWithTag}"
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
