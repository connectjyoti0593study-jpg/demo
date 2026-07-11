pipeline {
    agent any
    
    tools {
        dockerTool 'docker-cli' // This matches the name you set in Tools
    }

    environment {
        // Replace with your actual Docker Hub username
        DOCKER_HUB_USER = 'jyoti0593study'
        IMAGE_NAME      = 'spring-boot-app'
        IMAGE_TAG       = "${BUILD_NUMBER}"
    }

    stages {
        stage('Clone Repository') {
            steps {
                checkout scm
            }
        }

        stage('Build Artifact') {
            agent {
                docker {
                    image 'maven:3.9.16-eclipse-temurin-21'
                    // Reuses the local maven repository cache to speed up builds
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }


        stage('Build & Push Docker Image') {
            steps {
                script {
                    // Logs into Docker Hub and builds/pushes the image
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
                        sh "docker build -t ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} ."
                        sh "docker build -t ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest ."
                        sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                        sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
                    }
                }
            }
        }

        stage('Deploy to Container') {
            steps {
                // Stops the existing container if it exists, removes it, and runs the new one
                sh "docker stop ${IMAGE_NAME} || true"
                sh "docker rm ${IMAGE_NAME} || true"
                sh "docker run -d -p 8080:8080 --name ${IMAGE_NAME} ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
            }
        }
    }
}
