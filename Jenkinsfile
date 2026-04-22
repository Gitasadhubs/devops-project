pipeline {
    agent any

    environment {
        IMAGE_NAME = "devops-app"
        CONTAINER_NAME = "app"
        PORT = "8083"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'git@github.com:Gitasadhubs/devops-project.git',
                credentialsId: 'github-ssh'
            }
        }

        stage('Build Maven') {
            steps {
                sh '''
                echo "Building Maven project..."

                docker run --rm \
                -v $WORKSPACE/app:/app \
                -w /app \
                maven:3.9.6-eclipse-temurin-17 \
                mvn clean package -DskipTests
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                echo "Building Docker image..."

                docker build -t $IMAGE_NAME ./app
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                echo "Deploying application..."

                docker stop $CONTAINER_NAME || true
                docker rm $CONTAINER_NAME || true

                docker run -d \
                --name $CONTAINER_NAME \
                -p 8083:8080 \
                $IMAGE_NAME
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                echo "Waiting for app..."

                sleep 20

                curl -f http://localhost:${PORT}/ || exit 1

                echo "Application is running"
                '''
            }
        }

        stage('Show URL') {
            steps {
                echo "================================="
                echo "🚀 APP LIVE: http://localhost:8083"
                echo "================================="
            }
        }
    }

    post {
        success {
            echo "✅ Deployment SUCCESS"
        }

        failure {
            echo "❌ Deployment FAILED"
        }

        always {
            sh 'docker ps || true'
        }
    }
}
