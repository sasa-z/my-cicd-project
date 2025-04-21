pipeline {
    agent any
    
    tools {
        nodejs 'NodeJS 7.8.0'
    }
    
    environment {
        // Dinamički postavite port na osnovu imena grane
        PORT = "${BRANCH_NAME == 'main' ? '3000' : '3001'}"
        // Postavite ime Docker image-a na osnovu imena grane
        DOCKER_IMAGE_NAME = "node${BRANCH_NAME.toLowerCase()}:v1.0"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh "echo 'Trenutna grana: ${BRANCH_NAME}'"
                sh "echo 'Port koji će biti korišćen: ${PORT}'"
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
                    // Kreirajte Dockerfile za aplikaciju
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
            // Zaustavite i obrišite SAMO kontejnere za TRENUTNI environment
            sh "docker ps -q --filter ancestor=${DOCKER_IMAGE_NAME} | xargs -r docker stop"
            sh "docker ps -a -q --filter ancestor=${DOCKER_IMAGE_NAME} | xargs -r docker rm"
            
            // Pokrenite novi kontejner
            sh "docker run -d -p ${PORT}:${PORT} ${DOCKER_IMAGE_NAME}"
            
            echo "Aplikacija je uspešno deployed! Dostupna je na http://localhost:${PORT}"
        }
    }
}
}