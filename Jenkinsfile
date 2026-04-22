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

        stage('Verify Workspace') {
            steps {
                sh '''
                echo "===== WORKSPACE ====="
                pwd
                echo "WORKSPACE: $WORKSPACE"

                echo "Files:"
                ls -lah

                echo "Searching pom.xml..."
                find $WORKSPACE -name pom.xml
                '''
            }
        }

        stage('Build Maven (Docker)') {
            steps {
                sh '''
                echo "===== BUILDING JAR ====="

                docker run --rm \
                -v $WORKSPACE:/workspace \
                -w /workspace \
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
                -w /workspace \
                maven:3.9.6-eclipse-temurin-17 \
                mvn test
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                echo "===== BUILDING IMAGE ====="

                docker build -t $IMAGE_NAME .
                '''
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                echo "===== DEPLOYING ====="

                docker compose down || true
                docker compose up -d --build
                '''
            }
        }

        stage('Wait For Startup') {
            steps {
                sh '''
                echo "===== WAITING ====="
                sleep 30
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                echo "===== HEALTH CHECK ====="

                curl -f http://localhost:${PORT}/ || exit 1

                echo "Application is UP"
                '''
            }
        }

        stage('Show URL') {
            steps {
                sh '''
                echo "=================================="
                echo "🚀 APPLICATION RUNNING"
                echo "URL: http://localhost:${PORT}"
                echo "=================================="
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
            echo "===== CONTAINERS ====="
            docker ps || true
            '''
        }
    }
}        }

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
