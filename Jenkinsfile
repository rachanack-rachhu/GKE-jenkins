pipeline {
    agent any

    environment {
        PROJECT_ID = 'crested-polygon-472204-n5'          // 🔹 Replace with your actual GCP project ID
        REGION = 'us-central1'                      // 🔹 Same region where your repo and cluster exist
        REPO_NAME = 'flask-repo'                    // 🔹 Artifact Registry repo name
        IMAGE_NAME = 'flask-jenkins-app'
        IMAGE_TAG = 'latest'
        CLUSTER_NAME = 'jenkins-gke'                // 🔹 Your GKE cluster name
        CLUSTER_ZONE = 'us-central1-a'              // 🔹 Your GKE cluster zone
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/rachanack-rachhu/GKE-jenkins.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                    echo "🔧 Building Docker image..."
                    docker build -t $REGION-docker.pkg.dev/$PROJECT_ID/$REPO_NAME/$IMAGE_NAME:$IMAGE_TAG .
                    '''
                }
            }
        }

        stage('Push to Artifact Registry') {
            steps {
                script {
                    sh '''
                    echo "🔐 Authenticating Docker with Artifact Registry..."
                    gcloud auth configure-docker $REGION-docker.pkg.dev -q

                    echo "🚀 Pushing image to Artifact Registry..."
                    docker push $REGION-docker.pkg.dev/$PROJECT_ID/$REPO_NAME/$IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy to GKE') {
            steps {
                script {
                    sh '''
                    echo "☸️ Deploying to GKE cluster..."
                    gcloud container clusters get-credentials $CLUSTER_NAME --zone $CLUSTER_ZONE --project $PROJECT_ID

                    # Replace placeholder image in deployment YAML
                    sed -i "s|IMAGE_PLACEHOLDER|$REGION-docker.pkg.dev/$PROJECT_ID/$REPO_NAME/$IMAGE_NAME:$IMAGE_TAG|g" k8s/deploy.yaml

                    # Apply deployment to GKE
                    kubectl apply -f k8s/deploy.yaml

                    echo "✅ Deployment complete!"
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "🎉 CI/CD pipeline finished."
        }
    }
}
