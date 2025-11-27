pipeline {
    agent any

    environment {
        IMAGE_NAME = "nodejs-jenkins-sample-app"
        IMAGE_TAG  = "${env.BUILD_NUMBER}"
        FULL_IMAGE = "${IMAGE_NAME}:${IMAGE_TAG}"
        CONTAINER_NAME = "nodejs-jenkins-sample-app-container"
    }

    stages {
        stage('Checkout') {
            steps {
                // Récupère le code du repo configuré dans le job Jenkins
                checkout scm
            }
        }

        stage('Install dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                // Adapte si ton projet a une autre commande, par ex. "npm run test"
                sh 'npm test'
            }
        }

        stage('Build Docker image') {
            steps {
                sh "docker build -t ${FULL_IMAGE} ."
            }
        }

        stage('Deploy container') {
            steps {
                // Stop & remove l’ancien conteneur si présent
                sh "docker stop ${CONTAINER_NAME} || true"
                sh "docker rm ${CONTAINER_NAME} || true"

                // Lancer le conteneur sur localhost:3000
                sh "docker run -d --name ${CONTAINER_NAME} -p 3000:3000 ${FULL_IMAGE}"
            }
        }
    }

    post {
        always {
            echo "Build number: ${env.BUILD_NUMBER}"
            echo "Docker image: ${FULL_IMAGE}"
        }
    }
}
