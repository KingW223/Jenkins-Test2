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

        // ⚙️ Étape 2 : Configuration du Webhook SonarQube → Jenkins
        stage('Configure SonarQube Webhook') {
            steps {
                script {
                    echo "Configuration du webhook SonarQube vers Jenkins..."
                    sh '''
                        curl -u $SONAR_ADMIN_TOKEN: -X POST "http://sonarqube:9000/api/webhooks/create" \
                            -d "name=Jenkins_QualityGate" \
                            -d "url=http://jenkins2:9090/sonarqube-webhook/" \
                        || echo "⚠️  Webhook déjà existant ou erreur ignorée."
                    '''
                }
            }
        }

        // 🔍 Étape 3 : Analyse de la qualité du code avec SonarQube
        stage('SonarQube Analysis') {
            steps {
                echo 'Analyse du code avec SonarQube...'
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        sonar-scanner \
                            -Dsonar.projectKey=Jenkins-Test2 \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=http://sonarqube:9000 \
                            -Dsonar.login=$SONAR_ADMIN_TOKEN
                    '''
                }
            }
        }

        // ✅ Étape 4 : Vérification du Quality Gate
        stage('Quality Gate') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate(abortPipeline: false)
                        echo "Quality Gate status: ${qg.status}"
                        if (qg.status != 'OK') {
                            echo "⚠️  Attention: Quality Gate en erreur, le pipeline continue malgré tout."
                        }
                    }
                }
            }
        }

        // 🔑 Étape 5 : Connexion à Docker Hub
        stage('Login to DockerHub') {
            steps {
                echo 'Connexion à Docker Hub...'
                withCredentials([usernamePassword(credentialsId: 'king-hub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                }
            }
        }

        // 🛠️ Étape 6 : Construction de l’image backend
        stage('Build Backend Image') {
            steps {
                echo 'Construction de l’image backend...'
                sh 'docker build -t $DOCKER_HUB_REPO/backend:latest ./mon-projet-express'
            }
        }

        // 🛠️ Étape 7 : Construction de l’image frontend
        stage('Build Frontend Image') {
            steps {
                echo 'Construction de l’image frontend...'
                sh 'docker build -t $DOCKER_HUB_REPO/frontend:latest ./'
            }
        }

        // 📤 Étape 8 : Push des images vers Docker Hub
        stage('Push Images') {
            steps {
                echo 'Envoi des images vers Docker Hub...'
                sh '''
                    docker push $DOCKER_HUB_REPO/backend:latest
                    docker push $DOCKER_HUB_REPO/frontend:latest
                '''
            }
        }

        // 🚀 Étape 9 : Déploiement via Docker Compose
        stage('Deploy with Docker Compose') {
            steps {
                echo 'Déploiement via Docker Compose...'
                sh 'docker compose up -d'
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
            sh '''
                docker container prune -f
                docker image prune -f
                docker logout
            '''
        }
    }
}
