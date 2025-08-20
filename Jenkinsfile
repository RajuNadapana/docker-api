pipeline {
    agent any

    tools {
        jdk 'JDK-21'
        maven 'Maven-3.8.4'
    }

    environment {
        SONARQUBE_SERVER   = 'Sonar Cloud'
        SONAR_PROJECT_KEY  = 'docker-api'
        SONAR_PROJECT_NAME = 'docker-api'
        SONAR_ORGANIZATION = 'rajunadapana'

        APP_NAME              = 'docker-api'
        DOCKER_IMAGE          = "rajunadapana/${APP_NAME}"
        CONTAINER_NAME        = 'docker-api'
        APP_PORT              = '9595'
        DOCKERHUB_CREDENTIALS = 'docker-credentials'
        HOST_PORT_MAPPING     = '9595:9595'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Cloning repository...'
                git branch: 'main', url: 'https://github.com/RajuNadapana/docker-api.git'
            }
        }

        stage('Build') {
            steps {
                echo 'Building project with Maven...'
                bat 'mvn clean install -B'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                bat 'mvn test -B'
            }
            post {
                always {
                    junit '/target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarCloud Analysis') {
            steps {
                echo 'Running SonarCloud analysis...'
                withSonarQubeEnv('Sonar Cloud') {
                    withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                        bat """
                            mvn sonar:sonar ^
                            -Dsonar.projectKey=%SONAR_PROJECT_KEY% ^
                            -Dsonar.organization=%SONAR_ORGANIZATION% ^
                            -Dsonar.projectName=%SONAR_PROJECT_NAME% ^
                            -Dsonar.host.url=https://sonarcloud.io ^
                            -Dsonar.login=%SONAR_TOKEN%
                        """
                    }
                }
            }
        }

        /*
        stage('Quality Gate Check') {
            steps {
                echo 'Waiting 30 seconds to ensure SonarCloud processed the report...'
                sleep(time: 30, unit: 'SECONDS')
                timeout(time: 10, unit: 'MINUTES') {
                    echo 'Checking SonarCloud Quality Gate...'
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        */

        stage('Build Docker Image') {
            steps {
                script { env.IMAGE_TAG = "${env.BUILD_NUMBER}" }
                bat """
                    echo Building Docker image: %DOCKER_IMAGE%:%IMAGE_TAG%
                    docker build --pull -t %DOCKER_IMAGE%:%IMAGE_TAG% -t %DOCKER_IMAGE%:latest .
                """
            }
        }

        stage('Docker Push Image') {
            steps {
                echo "Logging into Docker registry and pushing image..."
                withCredentials([usernamePassword(
                    credentialsId: DOCKERHUB_CREDENTIALS,
                    usernameVariable: 'DOCKERHUB_USERNAME',
                    passwordVariable: 'DOCKERHUB_PASSWORD'
                )]) {
                    bat """
                        echo %DOCKERHUB_PASSWORD% | docker login -u %DOCKERHUB_USERNAME% --password-stdin
                        docker push %DOCKER_IMAGE%:%IMAGE_TAG%
                        docker push %DOCKER_IMAGE%:latest
                        docker logout || true
                    """
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                echo "Running Docker container..."
                bat """
                    docker stop %CONTAINER_NAME% || true
                    docker rm %CONTAINER_NAME% || true
                    docker run -d --name %CONTAINER_NAME% -p %HOST_PORT_MAPPING% %DOCKER_IMAGE%:latest
                """
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed with status: %BUILD_STATUS%'
            cleanWs()
        }
        success {
            echo 'Build, tests, SonarCloud analysis, Docker push, and container run were successful!'
        }
        failure {
            echo 'Pipeline failed! Check logs for details.'
        }
    }
}
