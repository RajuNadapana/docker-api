pipeline {
    agent any

    environment {
        // Add any environment variables you need here
        DOCKER_IMAGE = "rajunadapana/docker-api:latest"
        CONTAINER_NAME = "docker-api"
        HOST_PORT = "9596"
        CONTAINER_PORT = "9595"
    }

    stages {

        // Stage: Checkout code
        stage('Checkout') {
            steps {
                git 'https://github.com/RajuNadapana/docker-api.git'
            }
        }

        // Stage: Build Maven project
        stage('Build Maven Project') {
            steps {
                sh 'mvn clean install -B -DskipTests'
            }
        }

        // ------------------------------
        // Skipping SonarCloud Analysis
        // ------------------------------
        /*
        stage('SonarCloud Analysis') {
            steps {
                withSonarQubeEnv('SonarCloud') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        */

        // Stage: Build Docker image
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        // Stage: Run Docker container
        stage('Run Docker Container') {
            steps {
                // Stop & remove container if it exists
                sh "docker rm -f ${CONTAINER_NAME} || true"
                
                // Run container
                sh "docker run -d -p ${HOST_PORT}:${CONTAINER_PORT} --name ${CONTAINER_NAME} ${DOCKER_IMAGE}"
            }
        }
    }

    post {
        always {
            echo "Pipeline completed."
        }
        success {
            echo "Pipeline succeeded!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}
