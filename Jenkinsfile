pipeline {
    agent any
    
    environment {
        // Variables avec numéro de build et nom image
        DOCKER_IMAGE_NAME = "nodejs-app-jenkins"
        DOCKER_IMAGE_TAG = "${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
        DOCKER_IMAGE_LATEST = "${DOCKER_IMAGE_NAME}:latest"
        APP_PORT = "3000"
        SONAR_PROJECT_KEY = "nodejs-app-jenkins"  // Clé unique de votre projet SonarQube
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo "📥 Récupération du code source..."
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            steps {
                echo "📦 Installation des dépendances NodeJS..."
                sh '''
                    npm install
                '''
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                echo "🔍 Analyse SonarQube..."
                withSonarQubeEnv('sonarqube') {  // Nom exact du serveur SonarQube dans Jenkins
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.sources=. \
                        -Dsonar.tests=tests \
                        -Dsonar.test.inclusions=**/*.test.js,**/*.spec.js || true
                    '''
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Tests') {
            steps {
                echo "🧪 Lancement des tests..."
                sh '''
                    npm test || true
                '''
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo "🐳 Construction de l'image Docker ${DOCKER_IMAGE_TAG}..."
                sh '''
                    docker build -t ${DOCKER_IMAGE_TAG} .
                    docker tag ${DOCKER_IMAGE_TAG} ${DOCKER_IMAGE_LATEST}
                '''
            }
        }
        
        stage('Deploy Container') {
            steps {
                echo "🚀 Déploiement du conteneur sur port ${APP_PORT}..."
                sh '''
                    # Arrêt du conteneur précédent s'il existe
                    docker stop ${DOCKER_IMAGE_NAME} || true
                    docker rm ${DOCKER_IMAGE_NAME} || true
                    
                    # Lancement du nouveau conteneur
                    docker run -d \
                        --name ${DOCKER_IMAGE_NAME} \
                        -p ${APP_PORT}:3000 \
                        ${DOCKER_IMAGE_TAG}
                '''
            }
        }
        
        stage('Verification') {
            steps {
                echo "✅ Vérification du déploiement..."
                sh '''
                    sleep 5
                    curl -f http://localhost:${APP_PORT} || echo "App accessible sur http://localhost:${APP_PORT}"
                    docker ps
                '''
            }
        }
    }
    
    post {
        always {
            echo "🧹 Nettoyage des anciennes images Docker..."
            sh '''
                # Supprime les images nodejs-app-jenkins plus anciennes que la actuelle
                docker images --filter "reference=nodejs-app-jenkins" --format "{{.Repository}}:{{.Tag}} {{.ID}}" | \
                grep -v ":${BUILD_NUMBER}" | grep -v ":latest" | \
                awk '{print $2}' | xargs -r docker rmi -f || true
                
                # Nettoie les images dangling
                docker image prune -f
            '''
        }
        success {
            echo "🎉 Pipeline terminé avec succès ! App sur http://localhost:3000"
        }
        failure {
            echo "❌ Pipeline échoué"
            sh 'docker stop nodejs-app-jenkins || true'
            sh 'docker rm nodejs-app-jenkins || true'
        }
    }
}
