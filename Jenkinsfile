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
                docker run --rm \
                -v $WORKSPACE/app:/build \
                -w /build \
                maven:3.9.6-eclipse-temurin-17 \
                mvn clean package -DskipTests
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME ./app'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                docker compose down || true
                docker compose up -d --build
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                echo "Waiting for app..."
                sleep 20

                curl -f http://localhost:8083/ || exit 1
                echo "APP IS RUNNING"
                '''
            }
        }

        stage('App URL') {
            steps {
                echo "===================================="
                echo "🚀 APP LIVE: http://localhost:8083"
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
    }
}
