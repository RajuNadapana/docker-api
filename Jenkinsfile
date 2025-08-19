pipeline {
    agent any
    tools {
        jdk 'jdk-21'
        maven 'maven-3.8.4'
    }
    environment {
        SONARQUBE_SERVER = 'SonarCloud'
        SONAR_PROJECT_KEY = 'docker-api'
        SONAR_PROJECT_NAME = 'docker-api'
        SONAR_ORGANIZATION = 'rajunadapana'
        DOCKER_IMAGE = 'rajunadapana/docker-api'
        CONTAINER_NAME = 'docker-api'
        APP_PORT = '9595'
        DOCKERHUB_CREDENTIALS = 'your-dockerhub-credentials-id'
        HOST_PORT_MAPPING = '9596:9595'
        // Make sure to create a Jenkins secret text credential for SONAR_TOKEN if using -Dsonar.token
        SONAR_TOKEN = credentials('your-sonarcloud-token-id')
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/RajuNadapana/docker-api.git'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean install -B -DskipTests'
            }
        }
        stage('SonarCloud Analysis') {
            steps {
                withSonarQubeEnv('SonarCloud') {
                    sh """
                        mvn sonar:sonar \\
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \\
                        -Dsonar.organization=${SONAR_ORGANIZATION} \\
                        -Dsonar.projectName=${SONAR_PROJECT_NAME} \\
                        -Dsonar.host.url=https://sonarcloud.io \\
                        -Dsonar.token=${SONAR_TOKEN}
                    """
                }
            }
        }
        stage('Quality Gate Check') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    env.IMAGE_TAG = "${env.BUILD_NUMBER}"
                }
                sh "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} -t ${DOCKER_IMAGE}:latest ."
            }
        }
        stage('Docker Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKERHUB_CREDENTIALS}",
                    usernameVariable: 'DOCKERHUB_USERNAME',
                    passwordVariable: 'DOCKERHUB_PASSWORD'
                )]) {
                    sh '''
                        echo "$DOCKERHUB_PASSWORD" | docker login -u "$DOCKERHUB_USERNAME" --password-stdin
                        docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
                        docker push ${DOCKER_IMAGE}:latest
                        docker logout || true
                    '''
                }
            }
        }
        stage('Run Docker Container') {
            steps {
                sh "docker rm -f ${CONTAINER_NAME} || true"
                sh "docker run -d -p ${HOST_PORT_MAPPING} --name ${CONTAINER_NAME} ${DOCKER_IMAGE}:latest"
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
