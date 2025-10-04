pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = 'kingwest1'
    }

    triggers {
        // Pour que le pipeline démarre quand le webhook est reçu
        GenericTrigger(
            genericVariables: [
                [key: 'ref', value: '$.ref'],
                [key: 'pusher_name', value: '$.pusher.name'],
                [key: 'commit_message', value: '$.head_commit.message']
            ],
            causeString: 'Push par $pusher_name sur $ref: "$commit_message"',
            token: 'mysecret',
            printContributedVariables: true,
            printPostContent: true
        )
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

        stage('Clean Docker') {
            steps {
                sh 'docker container prune -f'
                sh 'docker image prune -f'
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                sh 'docker compose up -d'
            }
        }
    }

    
    post {
        success {
            emailext(
                subject: "Build SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Pipeline réussi\nDétails : ${env.BUILD_URL}",
                to: "naziftelecom2@gmail.com"
            )
        }
        failure {
            emailext(
                subject: "Build FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Le pipeline a échoué\nDétails : ${env.BUILD_URL}",
                to: "naziftelecom2@gmail.com"
            )
        }

        always {
            sh 'docker logout'
        }
    }

}
