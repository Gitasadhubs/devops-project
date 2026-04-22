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

        stage('Build Maven (Docker)') {
            steps {
                sh '''
                echo "===== BUILDING MAVEN PROJECT ====="

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
                echo "===== BUILDING DOCKER IMAGE ====="

                docker build -t $IMAGE_NAME ./app
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                echo "===== DEPLOYING CONTAINER ====="

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
                echo "===== HEALTH CHECK ====="

                sleep 20

                curl -f http://localhost:${PORT}/ || exit 1

                echo "APP IS RUNNING"
                '''
            }
        }

        stage('Show URL') {
            steps {
                echo "======================================"
                echo "🚀 APPLICATION LIVE"
                echo "URL: http://localhost:8083"
                echo "======================================"
            }
        }
    }

    post {
        success {
            echo "✅ PIPELINE SUCCESS"
        }

        failure {
            echo "❌ PIPELINE FAILED"
        }

        always {
            sh '''
            echo "===== CONTAINERS ====="
            docker ps || true
            '''
        }
    }
}
