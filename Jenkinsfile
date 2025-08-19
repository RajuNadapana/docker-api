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
                docker login -u %DOCKERHUB_USERNAME% -p %DOCKERHUB_PASSWORD%
                docker push %DOCKER_IMAGE%:%IMAGE_TAG%
                docker push %DOCKER_IMAGE%:latest
                docker logout
            """
        }
    }
}
