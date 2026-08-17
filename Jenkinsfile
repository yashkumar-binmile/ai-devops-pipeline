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
                    npm run test123 > pipeline.log 2>&1
                    TEST_STATUS=$?

                    cat pipeline.log

                    exit $TEST_STATUS
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

            echo '======================================'
            echo 'Pipeline failed - starting AI analysis'
            echo '======================================'

            withCredentials([
                string(
                    credentialsId: 'groq-api-key',
                    variable: 'GROQ_API_KEY'
                )
            ]) {

                sh '''
                    echo "Running AI analysis..."

                    python3 --version

                    python3 ai_analyze.py
                '''
            }
        }

        success {
            echo 'Pipeline completed successfully.'
        }
    }
}
