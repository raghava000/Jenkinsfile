pipeline {
    agent any

    // Define variables that we will use throughout the pipeline.
    environment {
        PROJECT_ID = 'ultra-sunset-446519-i4'
        REPO_NAME = 'my-devops-re-repo' // The name of your Artifact Registry repo
        IMAGE_NAME = 'my-first-app'
        LOCATION = 'asia-south1'
    }

    stages {
        stage('Checkout') {
            steps {
                // This is a real Jenkins step to checkout the code.
                checkout scm
            }
        }

        // We'll add a separate app directory with our Python files to this repo later.
        // For now, this stage will create them so the build works.
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
                // This is the magic. It tells Jenkins to use the Google credentials
                // we stored earlier for all the commands inside this block.
                withGoogleServiceAccount(credentialsId: env.PROJECT_ID) {
                    
                    // First, configure Docker to authenticate with gcloud.
                    sh "gcloud auth configure-docker ${LOCATION}-docker.pkg.dev --quiet"
                    
                    // Now, build the Docker image.
                    // We will tag it with the unique Jenkins BUILD_NUMBER.
                    sh "docker build -t ${LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${BUILD_NUMBER} ./app"
                    
                    // Finally, push the image to the registry.
                    sh "docker push ${LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${BUILD_NUMBER}"
                }
            }
        }
    }
}
