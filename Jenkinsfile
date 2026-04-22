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

        stage('Build & Package (Docker only)') {
            steps {
                sh '''
                echo "===== BUILDING IMAGE ====="

                docker build -t $IMAGE_NAME ./app
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                echo "===== DEPLOYING APP ====="

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
                echo "Waiting for app..."
                sleep 20

                curl -f http://localhost:${PORT}/ || exit 1
                echo "APP IS RUNNING"
                '''
            }
        }
    }

    post {
        success {
            echo "✅ DEPLOYMENT SUCCESS"
        }

        failure {
            echo "❌ DEPLOYMENT FAILED"
        }
    }
}
