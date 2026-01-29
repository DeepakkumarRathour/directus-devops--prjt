pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/DeepakkumarRathour/directus-devops--prjt.git'
            }
        }

        stage('Deploy Directus') {
            steps {
                sh '''
                cd /home/ubuntu/directus-prod
                docker-compose pull
                docker-compose up -d
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh 'curl -f http://127.0.0.1:8055'
            }
        }
    }
}

