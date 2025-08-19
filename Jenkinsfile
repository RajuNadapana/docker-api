pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "rajunadapana/docker-api:latest"
        CONTAINER_NAME = "docker-api"
        HOST_PORT = "9596"
        CONTAINER_PORT = "9595"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/RajuNadapana/docker-api.git'
            }
        }

        stage('Build Maven Project') {
            steps {
                bat 'mvn clean install -B -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t %DOCKER_IMAGE% ."
            }
        }

        stage('Run Docker Container') {
            steps {
                // Stop & remove container if it exists
                bat "docker rm -f %CONTAINER_NAME% || exit 0"
                
                // Run container
                bat "docker run -d -p %HOST_PORT%:%CONTAINER_PORT% --name %CONTAINER_NAME% %DOCKER_IMAGE%"
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
