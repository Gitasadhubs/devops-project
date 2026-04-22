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

        stage('Build Jar (Docker Maven)') {
            steps {
                dir('app') {
                    sh '''
                    docker run --rm \
                      -v $PWD:/app \
                      -w /app \
                      maven:3.9.6-eclipse-temurin-17 \
                      mvn clean package -DskipTests
                    '''
                }
            }
        }

        stage('Test (Docker Maven)') {
            steps {
                dir('app') {
                    sh '''
                    docker run --rm \
                      -v $PWD:/app \
                      -w /app \
                      maven:3.9.6-eclipse-temurin-17 \
                      mvn test
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME ./app'
            }
        }

        stage('Deploy (Compose)') {
            steps {
                sh '''
                echo "Stopping old stack..."
                docker compose down || true

                echo "Starting new stack..."
                docker compose up -d --build
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                echo "Waiting for app..."
                sleep 20

                curl -f http://localhost:8083 || exit 1
                '''
            }
        }

        stage('Show URL') {
            steps {
                echo "================================="
                echo "APP URL: http://localhost:8083"
                echo "================================="
            }
        }
    }

    post {
        success {
            echo "✅ Deployment Successful"
        }

        failure {
            echo "❌ Pipeline Failed"
        }
    }
}
