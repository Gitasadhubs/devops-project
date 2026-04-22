pipeline {
    agent any

    environment {
        IMAGE_NAME = "devops-app"
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

        stage('Verify Code') {
            steps {
                sh '''
                echo "===== WORKSPACE ====="
                pwd
                ls -lah

                echo "===== APP DIR ====="
                ls -lah app

                echo "===== POM CHECK ====="
                ls app/pom.xml
                '''
            }
        }

        stage('Build Maven (Docker)') {
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
                echo "Deploying container..."

                docker stop app || true
                docker rm app || true

                docker run -d \
                --name app \
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

                curl -f http://localhost:${PORT} || exit 1

                echo "APP IS RUNNING"
                '''
            }
        }

        stage('App URL') {
            steps {
                echo "===================================="
                echo "🚀 APP: http://localhost:8083"
                echo "===================================="
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
            sh 'docker ps || true'
        }
    }
}
