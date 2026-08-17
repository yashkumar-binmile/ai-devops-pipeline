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
                sh '''
                    npm install
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    /bin/bash -c 'npm test 2>&1 | tee pipeline.log'
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t ${IMAGE_NAME}:latest .
                '''
            }
        }
    }

    post {

        success {
            echo '''
======================================
PIPELINE SUCCESS
======================================
'''
        }

        failure {

            echo '''
======================================
PIPELINE FAILED
STARTING AI ANALYSIS
======================================
'''

            withCredentials([
                string(
                    credentialsId: 'groq-api-key',
                    variable: 'GROQ_API_KEY'
                )
            ]) {

                sh '''
                    echo "Collecting pipeline information..."

                    if [ ! -f pipeline.log ]; then
                        echo "No pipeline.log found"
                        exit 0
                    fi

                    echo "Sending failure information to AI..."

                    /bin/bash <<'EOF'

                    python3 - <<'PYTHON'

import os
from groq import Groq

api_key = os.environ.get("GROQ_API_KEY")

if not api_key:
    print("ERROR: GROQ_API_KEY is not available")
    exit(1)

client = Groq(api_key=api_key)

try:

    with open("pipeline.log", "r", errors="ignore") as f:
        log = f.read()

    # Prevent sending an extremely large Jenkins log
    log = log[-12000:]

    prompt = f"""
You are an AI DevOps assistant.

Analyze the following Jenkins CI/CD pipeline failure.

JENKINS PIPELINE LOG:
---------------------
{log}
---------------------

Provide:

1. Root cause
2. Failed stage
3. Exact error
4. Why it happened
5. Recommended fix
6. Correct command/configuration if possible

Keep the answer practical for a DevOps engineer.
"""

    response = client.chat.completions.create(
        model="llama-3.3-70b-versatile",
        messages=[
            {
                "role": "system",
                "content": "You are an expert DevOps CI/CD troubleshooting assistant."
            },
            {
                "role": "user",
                "content": prompt
            }
        ],
        temperature=0.2,
        max_tokens=1500
    )

    result = response.choices[0].message.content

    print("")
    print("======================================")
    print("AI DEVOPS ANALYSIS")
    print("======================================")
    print(result)
    print("======================================")

except Exception as e:

    print("AI analysis failed:")
    print(str(e))

PYTHON

EOF
                '''
            }
        }
    }
}
