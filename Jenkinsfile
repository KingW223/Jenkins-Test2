pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = 'kingwest1'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github-jenkins', url: 'https://github.com/ndiayekhardiata2024/Depot_Jenkins.git']])
            }
        }

        stage('Build Backend Image') {
            steps {
                script {
                    sh 'docker build -t $DOCKER_HUB_REPO/backend:latest ./mon-projet-express'
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                script {
                    sh 'docker build -t $DOCKER_HUB_REPO/frontend:latest ./'
                }
            }
        }


        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'jenkinaute', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
        }
    }
}



        

        stage('Push Images') {
            steps {
                script {
                    sh 'docker push $DOCKER_HUB_REPO/backend:latest'
                    sh 'docker push $DOCKER_HUB_REPO/frontend:latest'
                }
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                script {
                    sh 'docker compose down || true'
                    sh 'docker compose up -d'
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout'
        }
    }
}
