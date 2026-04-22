pipeline {
    agent any

    environment {
        IMAGE_NAME = "devops-app"
        CONTAINER_NAME = "app"
        PORT = "8083"
    }

    stages {

        stage('Checkout Code') {
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
                echo "Current Path:"
                pwd

                echo "Workspace:"
                echo $WORKSPACE

                echo "Workspace Files:"
                ls -lah $WORKSPACE

                echo "Searching pom.xml..."
                find $WORKSPACE -name pom.xml
                '''
            }
        }

        stage('Build Maven Project') {
            steps {
                sh '''
                echo "===== BUILDING JAR ====="

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

        stage('Build Docker Image') {
            steps {
                sh '''
                echo "===== BUILDING IMAGE ====="

                docker build -t $IMAGE_NAME ./app
                '''
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                echo "===== DEPLOYING APP ====="

                docker compose down || true
                docker compose up -d --build
                '''
            }
        }

        stage('Wait For Startup') {
            steps {
                sh '''
                echo "===== WAITING FOR STARTUP ====="
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

        stage('Show Application URL') {
            steps {
                sh '''
                echo "======================================"
                echo "🚀 APPLICATION DEPLOYED SUCCESSFULLY"
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
            echo "===== DOCKER STATUS ====="
            docker ps || true
            '''
        }
    }
}
