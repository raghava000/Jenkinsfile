pipeline {
    // This pipeline will run on any available Jenkins agent.
    agent any

    // --- Global Variables for the entire pipeline ---
    environment {
        PROJECT_ID        = 'ultra-sunset-446519-i4'
        REPO_NAME         = 'my-devops-re-repo'
        IMAGE_NAME        = 'my-first-app'
        LOCATION          = 'asia-south1'
        GCP_CREDENTIAL_ID = 'ultra-sunset-446519-i4'
        GKE_CLUSTER       = 'my-second-cluster' 
        GKE_ZONE          = 'us-central1-c'
        TARGET_NAMESPACE  = '' // This will be set dynamically based on the branch.
    }

    // --- The Assembly Line Stages ---
    stages {
        stage('Checkout') {
            steps {
                // Checks out the source code from the Git repository.
                checkout scm
            }
        }

        stage('Prepare Build Files') {
            steps {
                // This is a temporary step to create the app files needed for the build.
                // In a real project, these files would already be in your Git repository.
                sh '''
                    mkdir -p app
                    echo 'from flask import Flask' > app/app.py
                    echo 'app = Flask(__name__)' >> app/app.py
                    echo "@app.route('/')" >> app/app.py
                    echo 'def hello(): return "Hello from Jenkins Pipeline!"' >> app/app.py
                    echo 'if __name__ == "__main__": app.run(host="0.0.0.0", port=8080)' >> app/app.py
                    echo 'Flask' > app/requirements.txt
                    echo 'FROM python:3.9-slim' > app/Dockerfile
                    echo 'WORKDIR /app' >> app/Dockerfile
                    echo 'COPY requirements.txt .' >> app/Dockerfile
                    echo 'RUN pip install -r requirements.txt' >> app/Dockerfile
                    echo 'COPY . .' >> app/Dockerfile
                    echo 'CMD ["python", "app.py"]' >> app/Dockerfile
                '''
            }
        }

        stage('Build and Push Image') {
            steps {
                // This block securely loads our GCP key into a variable.
                withCredentials([string(credentialsId: GCP_CREDENTIAL_ID, variable: 'GCP_KEY_TEXT')]) {
                    // Write the secret to a temporary file.
                    writeFile(file: '/tmp/gcp-key.json', text: GCP_KEY_TEXT)

                    // Authenticate gcloud, then configure and use Docker.
                    sh "gcloud auth activate-service-account --key-file=/tmp/gcp-key.json"
                    sh "gcloud auth configure-docker ${LOCATION}-docker.pkg.dev --quiet"
                    sh "docker build -t ${LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${BUILD_NUMBER} ./app"
                    sh "docker push ${LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${BUILD_NUMBER}"
                    
                    // Clean up the secret file immediately.
                    sh "rm /tmp/gcp-key.json"
                }
            }
        }

        // --- THIS IS THE NEW, UPGRADED STAGE FOR DAY 20 ---
        stage('Deploy and Verify') {
            steps {
                script {
                    // This logic sets the target namespace based on the Git branch.
                    if (env.BRANCH_NAME == 'main') {
                        TARGET_NAMESPACE = 'prod'
                    } else {
                        TARGET_NAMESPACE = 'dev'
                    }
                    echo "Attempting deployment to namespace: ${TARGET_NAMESPACE}"
                }

                // The 'try...catch' block is our safety net.
                try {
                    // --- This is the "Risky Action" ---
                    // We will TRY to run everything in this block.
                    withCredentials([string(credentialsId: GCP_CREDENTIAL_ID, variable: 'GCP_KEY_TEXT')]) {
                        writeFile(file: '/tmp/gcp-key.json', text: GCP_KEY_TEXT)
                        sh "gcloud auth activate-service-account --key-file=/tmp/gcp-key.json"
                        sh "gcloud container clusters get-credentials ${GKE_CLUSTER} --zone=${GKE_ZONE}"
                        
                        // 1. THE DEPLOYMENT: Tell Kubernetes to update the image.
                        // For the Day 20 challenge, we add "-broken" to intentionally make it fail.
                        sh "kubectl set image deployment/my-web-app nginx=${LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${BUILD_NUMBER}-broken --namespace=${TARGET_NAMESPACE}"
                        
                        // 2. THE VERIFICATION: This command waits for the deployment to succeed.
                        // If the new pods crash or get stuck, this command will fail, triggering the 'catch' block.
                        sh "kubectl rollout status deployment/my-web-app --namespace=${TARGET_NAMESPACE}"

                        sh "rm /tmp/gcp-key.json"
                    }
                } catch (any) {
                    // --- This is the "Emergency Rollback Procedure" ---
                    // This code ONLY runs if a command in the 'try' block failed.
                    echo "Deployment failed! Starting automatic rollback..."
                    
                    // We still need credentials to perform the rollback.
                    withCredentials([string(credentialsId: GCP_CREDENTIAL_ID, variable: 'GCP_KEY_TEXT')]) {
                        writeFile(file: '/tmp/gcp-key.json', text: GCP_KEY_TEXT)
                        sh "gcloud auth activate-service-account --key-file=/tmp/gcp-key.json"
                        sh "gcloud container clusters get-credentials ${GKE_CLUSTER} --zone=${GKE_ZONE}"

                        // 3. THE ROLLBACK: Tell Kubernetes to revert to the previous successful version.
                        sh "kubectl rollout undo deployment/my-web-app --namespace=${TARGET_NAMESPACE}"
                        
                        sh "rm /tmp/gcp-key.json"
                    }

                    // 4. Officially fail the pipeline build.
                    error "Pipeline failed and deployment has been rolled back."
                }
            }
        }
    }
}
