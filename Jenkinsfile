pipeline {
    agent any

    environment {
        PROJECT_ID = 'applied-pager-476808-j5'
        GCR_REGION = 'gcr.io'
        IMAGE_NAME = 'robotshop'
        CLUSTER_NAME = 'gke-cluster'
        CLUSTER_ZONE = 'us-central1-a'
        HELM_CHART_PATH = 'K8s/helm'
        CREDENTIALS_ID = 'gcp-service-account-key' // Jenkins credential for GCP JSON key
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/TCRDINSEH/three-tier-architecture-demo.git'
            }
        }

        stage('Authenticate GCP') {
            steps {
                withCredentials([file(credentialsId: "${CREDENTIALS_ID}", variable: 'GCP_KEY')]) {
                    sh '''
                        gcloud auth activate-service-account --key-file=$GCP_KEY
                        gcloud config set project $PROJECT_ID
                        gcloud auth configure-docker ${GCR_REGION} --quiet
                    '''
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                    docker-compose build
                '''
            }
        }

        stage('Push Images to GCR') {
            steps {
                sh '''
                    for service in $(docker-compose config --services); do
                        IMAGE_TAG=${GCR_REGION}/$PROJECT_ID/${IMAGE_NAME}-$service:${BUILD_NUMBER}
                        docker tag robotshop_${service} $IMAGE_TAG
                        docker push $IMAGE_TAG
                    done
                '''
            }
        }

        stage('Deploy to GKE') {
            steps {
                sh '''
                    gcloud container clusters get-credentials $CLUSTER_NAME --zone $CLUSTER_ZONE --project $PROJECT_ID
                    helm upgrade --install robotshop ${HELM_CHART_PATH} \
                        --set image.repository=${GCR_REGION}/$PROJECT_ID/${IMAGE_NAME} \
                        --set image.tag=${BUILD_NUMBER}
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment successful 🚀'
        }
        failure {
            echo 'Pipeline failed ❌'
        }
    }
}
