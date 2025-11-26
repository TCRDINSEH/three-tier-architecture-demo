pipeline {
    agent any

    environment {
        PROJECT_ID = 'applied-pager-476808-j5'
        REGION = 'us-central1' // adjust to your GCP region
        REPO = "${REGION}-docker.pkg.dev/${PROJECT_ID}/robotshop" // Artifact Registry repo
        TAG = "${BUILD_NUMBER}"
        CLUSTER_NAME = 'gke-cluster'
        CLUSTER_ZONE = 'us-central1-a'
        HELM_CHART_PATH = 'GKE/helm'
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

         stage('Docker Build & Push Images') {
            steps {
                script {
                    def services = [
                        "mongodb","catalogue","user","cart","mysql",
                        "shipping","ratings","payment","dispatch","web"
                    ]
                    for (svc in services) {
                        sh """
                            docker build -t $REPO/rs-${svc}:$TAG ${svc}
                            docker push $REPO/rs-${svc}:$TAG
                        """
                    }
                    // Prebuilt images (redis, rabbitmq) are pulled from Docker Hub, no need to build/push
                }
            }
        }

            stage('Deploy to GKE') {
            steps {
                sh '''
                    gcloud container clusters get-credentials $CLUSTER_NAME --zone $CLUSTER_ZONE --project $PROJECT_ID
                    helm upgrade --install robotshop ${HELM_CHART_PATH} \
                        --set global.repo=$REPO \
                        --set global.tag=$TAG
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
