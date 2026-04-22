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

        stage('Locate Project') {
            steps {
                sh '''
                echo "Finding pom.xml..."
                find . -name pom.xml
                '''
            }
        }

        stage('Build Jar (Docker Maven)') {
            steps {
                dir('app') {
                    sh '''
                    echo "Building Maven project inside Docker..."

                    docker run --rm \
                      -v $PWD:/app \
                      -w /app \
                      maven:3.9.6-eclipse-temurin-17 \
                      mvn clean package -DskipTests
                    '''
                }
            }
        }

        stage('Test') {
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
                sh '''
                echo "Building Docker image..."
                docker build -t $IMAGE_NAME ./app
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                echo "Stopping old container..."
                docker stop $CONTAINER_NAME || true
                docker rm $CONTAINER_NAME || true

                echo "Starting new container..."
                docker run -d --name $CONTAINER_NAME \
                  -p 8083:8080 \
                  $IMAGE_NAME
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                echo "Waiting for application startup..."
                
                for i in {1..10}
                do
                  curl -f http://localhost:8083 && break
                  echo "Waiting... attempt $i"
                  sleep 5
                done
                '''
            }
        }

        stage('Show URL') {
            steps {
                echo "===================================="
                echo "🚀 APPLICATION LIVE"
                echo "URL: http://localhost:8083"
                echo "===================================="
            }
        }
    }

    post {
        success {
            echo "✅ Deployment Successful"
        }

        failure {
            echo "❌ Deployment Failed"
        }
    }
}
