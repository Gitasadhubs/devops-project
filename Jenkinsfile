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
                echo "===== WORKSPACE INFO ====="
                pwd
                echo "WORKSPACE = $WORKSPACE"

                echo "Root Files:"
                ls -lah

                echo "Workspace Files:"
                ls -lah $WORKSPACE

                echo "App Folder:"
                ls -lah $WORKSPACE/app || true
                '''
            }
        }

        stage('Build Maven (Docker)') {
            steps {
                sh '''
                echo "===== MAVEN BUILD ====="

                docker run --rm \
                -v $WORKSPACE:/workspace \
                -w /workspace/app \
                maven:3.9.6-eclipse-temurin-17 \
                mvn clean package -DskipTests
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                echo "===== RUNNING TESTS ====="

                docker run --rm \
                -v $WORKSPACE:/workspace \
                -w /workspace/app \
                maven:3.9.6-eclipse-temurin-17 \
                mvn test
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                echo "===== BUILDING IMAGE ====="

                docker build -t $IMAGE_NAME ./app
                '''
            }
        }

        stage('Deploy Stack') {
            steps {
                sh '''
                echo "===== DEPLOYING ====="

                docker compose down || true
                docker compose up -d --build
                '''
            }
        }

        stage('Wait For App') {
            steps {
                sh '''
                echo "===== WAITING FOR APP ====="
                sleep 30
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                echo "===== HEALTH CHECK ====="

                curl -f http://localhost:${PORT}/ || exit 1

                echo "Application is healthy"
                '''
            }
        }

        stage('Show URL') {
            steps {
                sh '''
                echo "======================================"
                echo "🚀 APPLICATION LIVE"
                echo "URL: http://localhost:${PORT}"
                echo "======================================"
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
            sh '''
            echo "===== CONTAINER STATUS ====="
            docker ps || true
            '''
        }
    }
}
