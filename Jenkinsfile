pipeline {
    agent any

    environment {
        APP_NAME = "app"
        IMAGE_NAME = "devops-app"
        PORT = "8083"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Gitasadhubs/devops-project.git',
                    credentialsId: 'github-token'
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
                sh 'docker build -t devops-app ./app'
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                echo "Stopping old container if exists..."

                docker ps -q --filter "name=$APP_NAME" | xargs -r docker stop || true
                docker ps -aq --filter "name=$APP_NAME" | xargs -r docker rm || true

                echo "Checking port $PORT usage..."
                if lsof -i :$PORT; then
                    echo "Port $PORT is already in use. Stopping process..."
                    fuser -k $PORT/tcp || true
                fi

                echo "Starting new container..."
                docker run -d --name $APP_NAME -p $PORT:8080 $IMAGE_NAME
                '''
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
