pipeline {
    agent any

    environment {
        COMPOSE_PROJECT_NAME = "directus-prod"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Stop Existing Containers') {
            steps {
                sh 'docker-compose down || true'
            }
        }

        stage('Start Directus') {
            steps {
                sh 'docker-compose up -d'
            }
        }

        stage('Wait for Directus') {
            steps {
                sh 'sleep 20'
            }
        }

        stage('Health Check') {
            steps {
                sh 'curl -f http://127.0.0.1:8055'
            }
        }
    }
}
