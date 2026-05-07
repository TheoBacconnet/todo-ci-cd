pipeline {
    agent {
        docker {
            image 'node:18-alpine'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        PROJECT_ID = 'todo-cicd-495614'
        REGION = 'europe-west1'
        REPO = 'todo-repo'
        IMAGE = 'todo-app'
        SERVICE = 'todo-service'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install & Test') {
            steps {
                sh 'npm install'
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO}/${IMAGE}:latest ."
            }
        }

        stage('Push to Artifact Registry') {
            steps {
                withCredentials([file(credentialsId: 'gcp-service-account', variable: 'GCP_KEY')]) {
                    sh """
                        gcloud auth activate-service-account --key-file=$GCP_KEY
                        gcloud auth configure-docker ${REGION}-docker.pkg.dev
                        docker push ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO}/${IMAGE}:latest
                    """
                }
            }
        }

        stage('Deploy to Cloud Run') {
            steps {
                withCredentials([file(credentialsId: 'gcp-service-account', variable: 'GCP_KEY')]) {
                    sh """
                        gcloud auth activate-service-account --key-file=$GCP_KEY
                        gcloud run deploy ${SERVICE} \
                            --image=${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO}/${IMAGE}:latest \
                            --platform=managed \
                            --region=${REGION} \
                            --allow-unauthenticated \
                            --project=${PROJECT_ID}
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}