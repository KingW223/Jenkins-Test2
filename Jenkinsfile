pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = 'kingwest1'
        SONAR_ADMIN_TOKEN = credentials('sonar-id')
    }

    triggers {
        // Déclenchement automatique via webhook GitHub
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

        // 🧩 Étape 1 : Récupération du code source
        stage('Checkout') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/main']],
                    extensions: [],
                    userRemoteConfigs: [[
                        credentialsId: 'king-github',
                        url: 'https://github.com/KingW223/Jenkins-Test2.git'
                    ]]
                )
            }
        }

        // 📦 Étape 2 : Installation des dépendances
        stage('Install dependencies') {
            steps {
                bat 'npm install'
            }
        }

        // 🔍 Étape 4 : Analyse SonarQube
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    script {
                        // Appel automatique du scanner configuré dans Jenkins
                        bat """
                            ${tool 'SonarScanner'}/bin/sonar-scanner.bat ^
                              -Dsonar.projectKey=express_mongo_react ^
                              -Dsonar.sources=. ^
                              -Dsonar.exclusions=**/node_modules/**,**/coverage/**,**/dist/**,**/build/** ^
                              -Dsonar.host.url=http://sonarqube:9000 ^
                              -Dsonar.token=%SONAR_ADMIN_TOKEN%
                        """
                    }
                }
            }
        }


        // ✅ Étape 5 : Quality Gate
        stage('Quality Gate') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    waitForQualityGate(abortPipeline: true)
                }
            }
        }

        // 🔑 Étape 6 : Connexion à Docker Hub
        stage('Login to DockerHub') {
            steps {
                echo 'Connexion à Docker Hub...'
                withCredentials([usernamePassword(credentialsId: 'king-hub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    bat '''
                        echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                    '''
                }
            }
        }

        // 🛠️ Étape 7 : Construction de l’image backend
        stage('Build Backend Image') {
            steps {
                echo 'Construction de l’image backend...'
                bat 'docker build -t %DOCKER_HUB_REPO%/backend:latest ./mon-projet-express'
            }
        }

        // 🛠️ Étape 8 : Construction de l’image frontend
        stage('Build Frontend Image') {
            steps {
                echo 'Construction de l’image frontend...'
                bat 'docker build -t %DOCKER_HUB_REPO%/frontend:latest ./'
            }
        }

        // 📤 Étape 9 : Push des images vers Docker Hub
        stage('Push Images') {
            steps {
                echo 'Envoi des images vers Docker Hub...'
                bat '''
                    docker push %DOCKER_HUB_REPO%/backend:latest
                    docker push %DOCKER_HUB_REPO%/frontend:latest
                '''
            }
        }

        // 🚀 Étape 10 : Déploiement via Docker Compose
        stage('Deploy with Docker Compose') {
            steps {
                echo 'Déploiement via Docker Compose...'
                bat 'docker compose up -d'
            }
        }
    }

    // 📬 Étapes post-pipeline
    post {
        success {
            emailext(
                subject: "✅ Build SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Pipeline réussi 🎉\nDétails : ${env.BUILD_URL}",
                to: "naziftelecom2@gmail.com"
            )
        }
        failure {
            emailext(
                subject: "❌ Build FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Le pipeline a échoué 😞\nDétails : ${env.BUILD_URL}",
                to: "naziftelecom2@gmail.com"
            )
        }
        always {
            echo 'Nettoyage des images et conteneurs Docker...'
            bat '''
                docker container prune -f
                docker image prune -f
                docker logout
            '''
        }
    }
}
