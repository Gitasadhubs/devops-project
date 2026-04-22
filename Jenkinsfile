pipeline {
    agent any

    environment {
        IMAGE_NAME = "devops-app"
        CONTAINER_NAME = "app"
        PORT = "8083"
        NETWORK_NAME = "devops-project_default"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'git@github.com:Gitasadhubs/devops-project.git',
                credentialsId: 'github-ssh'
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

        stage('Deploy Database') {
            steps {
                sh '''
                echo "===== STARTING DATABASE ====="

                docker network inspect $NETWORK_NAME >/dev/null 2>&1 || \
                docker network create $NETWORK_NAME

                docker stop db || true
                docker rm db || true

                docker run -d \
                --name db \
                --network $NETWORK_NAME \
                -e POSTGRES_DB=devops \
                -e POSTGRES_USER=postgres \
                -e POSTGRES_PASSWORD=postgres \
                -p 5432:5432 \
                postgres:15
                '''
            }
        }

        stage('Deploy App') {
            steps {
                sh '''
                echo "===== DEPLOYING APP ====="

                docker stop $CONTAINER_NAME || true
                docker rm $CONTAINER_NAME || true

                docker run -d \
                --name $CONTAINER_NAME \
                --network $NETWORK_NAME \
                -e SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/devops \
                -e SPRING_DATASOURCE_USERNAME=postgres \
                -e SPRING_DATASOURCE_PASSWORD=postgres \
                -p ${PORT}:8080 \
                $IMAGE_NAME
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                echo "===== WAITING FOR APP ====="

                for i in $(seq 1 30)
                do
                    curl -f http://localhost:${PORT}/ && break
                    echo "App not ready yet..."
                    sleep 10
                done

                curl -f http://localhost:${PORT}/
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

        always {
            sh '''
            echo "===== RUNNING CONTAINERS ====="
            docker ps || true
            '''
        }
    }
}
