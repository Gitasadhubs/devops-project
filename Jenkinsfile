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

        stage('Verify Workspace') {
            steps {
                sh '''
                echo "Workspace:"
                pwd
                ls -lah
                ls -lah $WORKSPACE/app
                '''
            }
        }

        stage('Build Maven (Docker)') {
            steps {
                sh '''
                echo "Building Maven project..."

                docker run --rm \
                -v $WORKSPACE:/workspace \
                -w /workspace/app \
                maven:3.9.6-eclipse-temurin-17 \
                mvn clean package -DskipTests
                '''
            }
        }

        stage('Docker Build') {
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
                echo "Stopping previous stack..."
                docker compose down || true

                echo "Starting new stack..."
                docker compose up -d --build
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                echo "Waiting for app startup..."
                sleep 30

                curl -f http://localhost:${PORT}/ || exit 1

                echo "Application is healthy"
                '''
            }
        }

        stage('App URL') {
            steps {
                sh '''
                echo "===================================="
                echo "🚀 APP LIVE: http://localhost:${PORT}"
                echo "===================================="
                '''
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
