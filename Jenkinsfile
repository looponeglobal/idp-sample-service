pipeline {
    agent any

    environment {
        AIRFLOW_URL     = "http://host.docker.internal:8080"
        AIRFLOW_CREDS   = credentials("airflow-api-credentials")
        SERVICE_NAME    = "idp-sample-service"
        GIT_COMMIT_HASH = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
    }

    stages {

        stage("Checkout") {
            steps {
                echo "Checking out code..."
                checkout scm
            }
        }
      

        stage("Install Dependencies") {
            steps {
                sh '''
                    # 1. Create a virtual environment directory without using the system's broken pip
                    python3 -m venv my_env --without-pip
                    
                    # 2. Download the standalone pip installer
                    curl -sS https://pypa.io -o get-pip.py
                    
                    # 3. Force-install pip directly INSIDE your new virtual environment folder
                    ./my_env/bin/python3 get-pip.py
                    
                    # 4. Use the specific virtual environment pip to download your dependencies
                    ./my_env/bin/pip install -r requirements.txt
                    
                    # 5. (Optional) Run your script using the environment's python runner
                    ./my_env/bin/python3 your_script.py
                '''
            }
        }

        stage("Run Tests") {
            steps {
                echo "Running tests..."
                sh "pytest test_app.py -v"
            }
        }

        stage("Trigger Airflow DAG") {
            steps {
                echo "Notifying Airflow to deploy ${SERVICE_NAME}:${GIT_COMMIT_HASH}..."
                sh """
                    curl -X POST "${AIRFLOW_URL}/api/v1/dags/provision_environment/dagRuns" \
                        -u "${AIRFLOW_CREDS_USR}:${AIRFLOW_CREDS_PSW}" \
                        -H "Content-Type: application/json" \
                        -d '{
                            "conf": {
                                "service_name": "${SERVICE_NAME}",
                                "version": "${GIT_COMMIT_HASH}",
                                "owner_team": "platform",
                                "owner_email": "team@example.com",
                                "environment": "dev",
                                "cloud_provider": "aws"
                            }
                        }'
                """
            }
        }

    }

    post {
        success {
            echo "Pipeline completed successfully"
        }
        failure {
            echo "Pipeline failed — check logs above"
        }
    }
}
