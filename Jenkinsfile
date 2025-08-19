// Jenkinsfile for Windows CMD
pipeline {

    agent any

    tools {
        jdk 'jdk-21'
        maven 'maven-3.8.4'
    }

    environment {
        // SonarQube configuration
        SONARQUBE_SERVER = 'SonarCloud'
        SONAR_PROJECT_KEY = 'ashish-panicker_simple-spring-api'
        SONAR_PROJECT_NAME = 'simple-spring-api'
        SONAR_ORGANIZATION = 'ashish-panicker'

        // Docker / Deploy
        APP_NAME              = 'simple-spring-api'
        DOCKER_IMAGE          = "ashishspanicker/${APP_NAME}"    
        CONTAINER_NAME        = 'simple-spring-api'
        APP_PORT              = '9595'                          
        DOCKERHUB_CREDENTIALS = 'docker-credentials'
        HOST_PORT_MAPPING     = '9595:9595'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Cloning repository...'
                git branch: 'main', url: 'https://github.com/ashish-panicker/simple-spring-api.git'
            }
        }

        stage('Build') {
            steps {
                echo 'Building with Maven...'
                bat 'mvn clean install'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                bat 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'Running SonarQube analysis...'
                withSonarQubeEnv(SONARQUBE_SERVER) {
                    bat """
                        mvn sonar:sonar ^
                        -Dsonar.projectKey=%SONAR_PROJECT_KEY% ^
                        -Dsonar.organization=%SONAR_ORGANIZATION% ^
                        -Dsonar.projectName=%SONAR_PROJECT_NAME%
                    """
                }
            }
            post {
                always {
                    echo 'SonarQube analysis completed.'
                }
            }
        }

        stage('Quality Gate Check') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    echo 'Waiting for SonarQube quality gate...'
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    env.IMAGE_TAG = "${env.BUILD_NUMBER}"
                }
                bat """
                    echo Building Docker image: %DOCKER_IMAGE%:%IMAGE_TAG%
                    docker build --pull -t %DOCKER_IMAGE%:%IMAGE_TAG% -t %DOCKER_IMAGE%:latest .
                """
            }
        }

        stage('Docker Push Image') {
            steps {
                echo "Logging into registry and pushing %DOCKER_IMAGE%:%IMAGE_TAG%"
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKERHUB_CREDENTIALS}",
                    usernameVariable: 'DOCKERHUB_USERNAME',
                    passwordVariable: 'DOCKERHUB_PASSWORD'
                )]) {
                    bat """
                        echo %DOCKERHUB_PASSWORD% | docker login -u %DOCKERHUB_USERNAME% --password-stdin
                        docker push %DOCKER_IMAGE%:%IMAGE_TAG%
                        docker push %DOCKER_IMAGE%:latest
                        docker logout
                    """
                }
            }
        }

    }

    post {
        always {
            echo 'Pipeline finished with status: ${currentBuild.currentResult}'
        }
    }

}
