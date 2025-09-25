pipeline {
    agent any

    environment {
        PROJECT_ID = 'ultra-sunset-446519-i4'
        REPO_NAME = 'my-devops-re-repo'
        IMAGE_NAME = 'my-first-app' // This is the image we are building
        LOCATION = 'asia-south1'
        GCP_CREDENTIAL_ID = 'ultra-sunset-446519-i4'
        GKE_CLUSTER = 'cluster1' // The name of your GKE cluster
        GKE_ZONE = 'us-central1-a' // The zone of your cluster
    }

    stages {
        // ... Checkout and Prepare Build Files stages are the same ...
        stage('Checkout') { /* ... */ }
        stage('Prepare Build Files') { /* ... */ }

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

        // --- THIS IS THE NEW STAGE ---
        stage('Deploy to GKE') {
            steps {
                // We use the same secure block to ensure we are authenticated
                withCredentials([string(credentialsId: GCP_CREDENTIAL_ID, variable: 'GCP_KEY_TEXT')]) {
                    
                    // Step 1: Authenticate gcloud again (good practice for clean stages)
                    sh "gcloud auth activate-service-account --key-file=/tmp/gcp-key.json"

                    // Step 2: Connect kubectl to our GKE cluster
                    sh "gcloud container clusters get-credentials ${GKE_CLUSTER} --zone=${GKE_ZONE}"

                    // Step 3: Use kubectl to update the running deployment's image
                    // This is the core "CD" command.
                    sh "kubectl set image deployment/my-web-app my-first-app=${LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${BUILD_NUMBER}"

                    // Step 4: Securely clean up the key file
                    sh "rm /tmp/gcp-key.json"
                }
            }
        }
    }
}
