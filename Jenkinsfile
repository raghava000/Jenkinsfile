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
        // NEW: This variable will hold the target namespace
        TARGET_NAMESPACE = '' 
    }

    stages {
        stage('Checkout') { /* ... same as before ... */ }
        stage('Prepare Build Files') { /* ... same as before ... */ }
        stage('Build and Push Image') { /* ... same as before ... */ }

        // --- THIS STAGE IS UPGRADED ---
        stage('Deploy to GKE') {
            steps {
                script {
                    // This is a Groovy script block. It allows for logic like if/else.
                    // Jenkins automatically provides the name of the branch being built
                    // in a special variable called env.BRANCH_NAME.
                    if (env.BRANCH_NAME == 'main') {
                        TARGET_NAMESPACE = 'prod'
                    } else if (env.BRANCH_NAME == 'dev') {
                        TARGET_NAMESPACE = 'dev'
                    } else {
                        // For any other branch, we'll just deploy to dev.
                        TARGET_NAMESPACE = 'dev'
                    }
                    echo "Deploying to namespace: ${TARGET_NAMESPACE}"
                }

                withCredentials([string(credentialsId: GCP_CREDENTIAL_ID, variable: 'GCP_KEY_TEXT')]) {
                    writeFile(file: '/tmp/gcp-key.json', text: GCP_KEY_TEXT)
                    sh "gcloud auth activate-service-account --key-file=/tmp/gcp-key.json"
                    sh "gcloud container clusters get-credentials ${GKE_CLUSTER} --zone=${GKE_ZONE}"

                    // IMPORTANT: We add the --namespace flag to our kubectl command
                    sh "kubectl set image deployment/my-web-app nginx=${LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${BUILD_NUMBER} --namespace=${TARGET_NAMESPACE}"
                    
                    sh "rm /tmp/gcp-key.json"
                }
            }
        }
    }
}
