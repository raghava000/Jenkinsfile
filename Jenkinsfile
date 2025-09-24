pipeline {
    agent any

    environment {
        PROJECT_ID = 'ultra-sunset-446519-i4'
        REPO_NAME = 'my-devops-re-repo'
        IMAGE_NAME = 'my-first-app'
        LOCATION = 'asia-south1'
        GCP_CREDENTIAL_ID = 'ultra-sunset-446519-i4' 
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

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
                // Use the 'string' provider for "Secret text" credentials.
                // This puts the entire JSON key into the GCP_KEY_TEXT environment variable.
                withCredentials([string(credentialsId: GCP_CREDENTIAL_ID, variable: 'GCP_KEY_TEXT')]) {
                    
                    // The gcloud command needs a file, not text. So we write the text to a temporary file.
                    // This is a standard, robust pattern.
                    sh "echo '${GCP_KEY_TEXT}' > /tmp/gcp-key.json"

                    // Step 1: Explicitly activate the Service Account using the temporary key file.
                    sh "gcloud auth activate-service-account --key-file=/tmp/gcp-key.json"

                    // Step 2: Configure Docker to use the now-activated gcloud credentials.
                    sh "gcloud auth configure-docker ${LOCATION}-docker.pkg.dev --quiet"
                    
                    // Step 3: Build the Docker image.
                    sh "docker build -t ${LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${BUILD_NUMBER} ./app"
                    
                    // Step 4: Push the image to the registry.
                    sh "docker push ${LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${BUILD_NUMBER}"

                    // Step 5: Clean up the temporary secret file.
                    sh "rm /tmp/gcp-key.json"
                }
            }
        }
    }
}
