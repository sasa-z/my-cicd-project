pipeline {
    agent any
    
    tools {
        nodejs 'NodeJS 7.8.0'
    }
    
    environment {
        // Dynamically set the port based on the branch name
        PORT = "${BRANCH_NAME == 'main' ? '3000' : '3001'}"
        // Set the Docker image name based on the branch name
        DOCKER_IMAGE_NAME = "node${BRANCH_NAME.toLowerCase()}:v1.0"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh "echo 'Current branch: ${BRANCH_NAME}'"
                sh "echo 'Port that will be used: ${PORT}'"
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Create Dockerfile for the application
                    writeFile file: 'Dockerfile', text: """
                        FROM node:7.8.0-alpine
                        WORKDIR /app
                        COPY . .
                        RUN npm install
                        ENV PORT=${PORT}
                        EXPOSE ${PORT}
                        CMD ["npm", "start"]
                    """
                    
                    // Build Docker image
                    sh "docker build -t ${DOCKER_IMAGE_NAME} ."
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    // Stop and remove ONLY containers for the CURRENT environment
                    sh "docker ps -q --filter ancestor=${DOCKER_IMAGE_NAME} | xargs -r docker stop"
                    sh "docker ps -a -q --filter ancestor=${DOCKER_IMAGE_NAME} | xargs -r docker rm"
                    
                    // Start a new container
                    sh "docker run -d -p ${PORT}:${PORT} ${DOCKER_IMAGE_NAME}"
                    
                    echo "Application successfully deployed! Available at http://localhost:${PORT}"
                }
            }
        }
    }
}
