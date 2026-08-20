pipeline {

    agent any

    environment {
        IMAGE_NAME = "ai-devops-demo"
    }

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
                    npm test 2>&1 | tee pipeline.log
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                '''
            }
        }
    }

    post {

        success {
            echo """
======================================
PIPELINE SUCCESS
======================================

Build Number: ${BUILD_NUMBER}
Docker Image: ${IMAGE_NAME}:${BUILD_NUMBER}
"""
        }

        failure {
            echo """
======================================
PIPELINE FAILED
======================================
"""
        }
    }
}
