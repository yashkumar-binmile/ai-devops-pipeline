pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh '''
                    set -o pipefail
                    npm test 2>&1 | tee pipeline.log
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t ai-devops-demo .'
            }
        }
    }

    post {

        failure {

            withCredentials([
                string(
                    credentialsId: 'groq-api-key',
                    variable: 'GROQ_API_KEY'
                )
            ]) {

                sh '''
                    python3 ai_analyze.py
                '''
            }
        }
    }
}
