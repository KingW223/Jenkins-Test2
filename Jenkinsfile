pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = 'kingwest1'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'king-github', url: 'https://github.com/KingW223/Jenkins-Test2.git']])
            }
        }

        stage('Build Backend Image') {
            steps {
                sh 'docker build -t $DOCKER_HUB_REPO/backend:latest ./mon-projet-express'
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh 'docker build -t $DOCKER_HUB_REPO/frontend:latest ./'
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'king-hub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                }
            }
        }

        stage('Push Images') {
            steps {
                sh 'docker push $DOCKER_HUB_REPO/backend:latest'
                sh 'docker push $DOCKER_HUB_REPO/frontend:latest'
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                sh 'docker compose up -d'
            }
        }
    }

    post {
        always {
            sh 'docker logout'
        }
    }
}
