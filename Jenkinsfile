pipeline {
    agent any

    tools {
        jdk 'JDK-21'          // Your Jenkins JDK installation
        maven 'Maven-3.8.4'   // Your Jenkins Maven installation
    }

    environment {
        // SonarCloud configuration
        SONARQUBE_SERVER   = 'Sonar Cloud'                     // Jenkins SonarCloud server name
        SONAR_PROJECT_KEY  = 'RajuNadapana_simple-emp-api'    // Your SonarCloud project key
        SONAR_PROJECT_NAME = 'simple-emp-api'                 // Project name
        SONAR_ORGANIZATION = 'rajunadapana'                   // Organization key (lowercase)

        // Docker / Deployment
        APP_NAME              = 'simple-emp-api'
        DOCKER_IMAGE          = "rajunadapana/${APP_NAME}"    
        CONTAINER_NAME        = 'simple-emp-api'
        APP_PORT              = '9595'                          
        DOCKERHUB_CREDENTIALS = 'docker-credentials'          // Jenkins DockerHub credentials ID
        HOST_PORT_MAPPING     = '9595:9595'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Cloning repository...'
                git branch: 'main', url: 'https://github.com/RajuNadapana/simple-emp-api.git'
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
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarCloud Analysis') {
            steps {
                echo 'Running SonarCloud analysis...'
                withSonarQubeEnv(SONARQUBE_SERVER) {
                    bat """
                        mvn sonar:sonar ^
                        -Dsonar.projectKey=%SONAR_PROJECT_KEY% ^
                        -Dsonar.organization=%SONAR_ORGANIZATION% ^
                        -Dsonar.projectName=%SONAR_PROJECT_NAME% ^
                        -Dsonar.host.url=https://sonarcloud.io
                    """
                }
            }
            post {
                always {
                    echo 'SonarCloud analysis completed.'
                }
            }
        }

        stage('Quality Gate Check') {
            steps {
                echo 'Waiting 10 seconds to ensure SonarCloud processed the report...'
                sleep(time: 10, unit: 'SECONDS')
                timeout(time: 10, unit: 'MINUTES') {
                    echo 'Checking SonarCloud Quality Gate...'
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
                echo "Logging into Docker registry and pushing image..."
                withCredentials([usernamePassword(
                    credentialsId: "%DOCKERHUB_CREDENTIALS%",
                    usernameVariable: 'DOCKERHUB_USERNAME',
                    passwordVariable: 'DOCKERHUB_PASSWORD'
                )]) {
                    bat '''
                        echo %DOCKERHUB_PASSWORD% | docker login -u %DOCKERHUB_USERNAME% --password-stdin
                        docker push %DOCKER_IMAGE%:%IMAGE_TAG%
                        docker push %DOCKER_IMAGE%:latest
                        docker logout || true
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed with status: %BUILD_STATUS%'
            cleanWs()
        }
        success {
            echo 'Build, tests, SonarCloud analysis, and Docker push were successful!'
        }
        failure {
            echo 'Pipeline failed! Check logs for details.'
        }
    }
}
