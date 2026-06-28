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

        steps {
        sh '''
            sudo apt-get update
            sudo apt-get install -y python3-pip
        '''
        }

        stage("Install Dependencies") {
            steps {
                echo "Installing Python dependencies..."
                sh "pip install -r requirements.txt"
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
