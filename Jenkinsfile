pipeline {
    agent any

    environment {
        APP_NAME = "java_app"
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

        stage('Build Jar') {
            steps {
                dir('app') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Test') {
            steps {
                dir('app') {
                    sh 'mvn test'
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
                sleep 15

                curl -f http://localhost:8083 || exit 1
                echo "App is UP"
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
