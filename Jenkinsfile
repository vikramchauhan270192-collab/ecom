pipeline {
    agent any

    environment {
        APP_NAME = "spring-boot-app"
        IMAGE_NAME = "spring-boot-app"
        DOCKERHUB_USER = "your_dockerhub_username"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/your-org/spring-boot-app.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh './mvnw clean test'
            }
        }

        stage('Package') {
            steps {
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t $DOCKERHUB_USER/$IMAGE_NAME:${BUILD_NUMBER} .
                docker tag $DOCKERHUB_USER/$IMAGE_NAME:${BUILD_NUMBER} \
                           $DOCKERHUB_USER/$IMAGE_NAME:latest
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $DOCKERHUB_USER/$IMAGE_NAME:${BUILD_NUMBER}
                    docker push $DOCKERHUB_USER/$IMAGE_NAME:latest
                    """
                }
            }
        }

        stage('Deploy (Docker)') {
            steps {
                sh """
                docker stop $APP_NAME || true
                docker rm $APP_NAME || true
                docker run -d \
                  -p 8080:8080 \
                  --name $APP_NAME \
                  $DOCKERHUB_USER/$IMAGE_NAME:latest
                """
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful!"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
