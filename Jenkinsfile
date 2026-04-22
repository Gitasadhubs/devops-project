pipeline {
    agent any

    environment {
        APP_NAME = "app"
        IMAGE_NAME = "devops-app"
        PORT = "8083"
        VERSION = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Gitasadhubs/devops-project.git'
            }
        }

        stage('Build') {
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
                sh "docker build -t $IMAGE_NAME:$VERSION ./app"
            }
        }

        stage('Deploy') {
            steps {
                sh """
                docker rm -f $APP_NAME || true
                docker run -d --name $APP_NAME -p $PORT:8080 $IMAGE_NAME:$VERSION
                """
            }
        }

        stage('Health Check') {
            steps {
                sh """
                sleep 10
                curl -f http://localhost:$PORT || exit 1
                """
            }
        }

        stage('Show Application URL') {
            steps {
                script {
                    def host = sh(script: "hostname -I | awk '{print \$1}'", returnStdout: true).trim()
                    echo "--------------------------------------------------"
                    echo "🚀 APPLICATION DEPLOYED SUCCESSFULLY"
                    echo "👉 URL: http://${host}:${PORT}"
                    echo "--------------------------------------------------"
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment SUCCESS"
        }

        failure {
            echo "❌ Deployment FAILED"
            sh 'docker logs app || true'
        }
    }
}
