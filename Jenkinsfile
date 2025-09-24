pipeline {
    agent any

    environment {
        PROJECT_ID = 'ultra-sunset-446519-i4' // Hardcode your Project ID
        REPO_NAME = 'my-devops-re-repo'
        IMAGE_NAME = 'my-first-app'
        LOCATION = 'asia-south1'
        // This is the ID you gave your credential in the Jenkins UI
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
                // This stage is the same as before, creating the app files
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
                // --- THIS IS THE NEW, ROBUST METHOD ---
                // Use the standard 'withCredentials' step to securely access our key file.
                // It will make the file available at the path stored in the 'GCP_KEY' variable.
                withCredentials([file(credentialsId: GCP_CREDENTIAL_ID, variable: 'GCP_KEY')]) {
                    
                    // Step 1: Explicitly activate the Service Account using the key file.
                    // This is the "brute force" authentication.
                    sh "gcloud auth activate-service-account --key-file=${GCP_KEY}"

                    // Step 2: Configure Docker to use the now-activated gcloud credentials.
                    sh "gcloud auth configure-docker ${LOCATION}-docker.pkg.dev --quiet"
                    
                    // Step 3: Build the Docker image.
                    sh "docker build -t ${LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${BUILD_NUMBER} ./app"
                    
                    // Step 4: Push the image to the registry.
                    sh "docker push ${LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${BUILD_NUMBER}"
                }
            }
        }
    }
}
