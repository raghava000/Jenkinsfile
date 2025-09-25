pipeline {
    agent any

    environment {
        PROJECT_ID = 'ultra-sunset-446519-i4'
        REPO_NAME = 'my-devops-re-repo'
        IMAGE_NAME = 'my-first-app'
        LOCATION = 'asia-south1'
        GCP_CREDENTIAL_ID = 'ultra-sunset-446519-i4'
        GKE_CLUSTER = 'my-second-cluster' 
        GKE_ZONE = 'us-central1-c'
    }

    stages {
        // Correctly defined Checkout stage
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // Correctly defined Prepare stage
        stage('Prepare Build Files') {
            steps {
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
                withCredentials([string(credentialsId: GCP_CREDENTIAL_ID, variable: 'GCP_KEY_TEXT')]) {
                    writeFile(file: '/tmp/gcp-key.json', text: GCP_KEY_TEXT)
                    sh "gcloud auth activate-service-account --key-file=/tmp/gcp-key.json"
                    sh "gcloud auth configure-docker ${LOCATION}-docker.pkg.dev --quiet"
                    sh "docker build -t ${LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${BUILD_NUMBER} ./app"
                    sh "docker push ${LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${BUILD_NUMBER}"
                    sh "rm /tmp/gcp-key.json"
                }
            }
        }

         stage('Deploy to GKE') {
            steps {
                withCredentials([string(credentialsId: GCP_CREDENTIAL_ID, variable: 'GCP_KEY_TEXT')]) {
                    
                    // --- THE FIX IS THIS ONE LINE ---
                    // Re-create the key file for this stage's scope.
                    writeFile(file: '/tmp/gcp-key.json', text: GCP_KEY_TEXT)

                    // The rest of the steps will now succeed because the file exists.
                    sh "gcloud auth activate-service-account --key-file=/tmp/gcp-key.json"
                    sh "gcloud container clusters get-credentials ${GKE_CLUSTER} --zone=${GKE_ZONE}"
                    sh "kubectl set image deployment/my-web-app nginx=${LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${BUILD_NUMBER}"
                    sh "rm /tmp/gcp-key.json"
                }
            }
        }
    }
}
