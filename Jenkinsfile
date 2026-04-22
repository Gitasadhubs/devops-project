pipeline {
    agent any

    environment {
        IMAGE_NAME = "devops-app"
        CONTAINER_NAME = "app"
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

        stage('Deploy') {
            steps {
                sh '''
                docker stop $CONTAINER_NAME || true
                docker rm $CONTAINER_NAME || true

                docker run -d --name $CONTAINER_NAME \
                  -p 8083:8080 \
                  $IMAGE_NAME
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                sleep 15
                curl -f http://localhost:8083 || exit 1
                '''
            }
        }

        stage('Show URL') {
            steps {
                echo "APP RUNNING: http://localhost:8083"
            }
        }
    }
}
