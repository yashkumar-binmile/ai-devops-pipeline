pipeline {

    agent any

    parameters {
        string(
            name: 'ROLLBACK_VERSION',
            defaultValue: '',
            description: 'Leave empty for normal deployment. Enter an old build number for rollback.'
        )
    }

    environment {
        IMAGE_NAME = "ai-devops-demo"
        CONTAINER_NAME = "ai-devops-demo"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install') {
            when {
                expression {
                    !params.ROLLBACK_VERSION?.trim()
                }
            }
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            when {
                expression {
                    !params.ROLLBACK_VERSION?.trim()
                }
            }
            steps {
                sh '''
                    npm test 2>&1 | tee pipeline.log
                '''
            }
        }

        stage('Docker Build') {
            when {
                expression {
                    !params.ROLLBACK_VERSION?.trim()
                }
            }
            steps {
                sh '''
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Deploy') {
            steps {
                script {

                    def version

                    if (params.ROLLBACK_VERSION?.trim()) {
                        version = params.ROLLBACK_VERSION

                        echo "======================================"
                        echo "ROLLBACK"
                        echo "Deploying: ${IMAGE_NAME}:${version}"
                        echo "======================================"

                    } else {
                        version = BUILD_NUMBER

                        echo "======================================"
                        echo "NORMAL DEPLOYMENT"
                        echo "Deploying: ${IMAGE_NAME}:${version}"
                        echo "======================================"
                    }

                    sh """
                        docker stop ${CONTAINER_NAME} || true
                        docker rm ${CONTAINER_NAME} || true

                        docker run -d \
                            --name ${CONTAINER_NAME} \
                            -p 8080:3000 \
                            ${IMAGE_NAME}:${version}
                    """
                }
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
