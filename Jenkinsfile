// This defines the entire assembly line.
pipeline {
    // Tell Jenkins to use any available "robotic arm" (agent) to run this job.
    agent any

    // This block defines all the sequential stages of our assembly line.
    stages {
        // Stage 1: Checkout Code
        stage('Checkout') {
            steps {
                // This step just prints a message to the console.
                echo 'Checking out code from version control...'
                // Later, we will add the actual 'git checkout' command here.
            }
        }

        // Stage 2: Build
        stage('Build') {
            steps {
                echo 'Building the application...'
                // Later, this is where we would run 'docker build'.
            }
        }

        // Stage 3: Test
        stage('Test') {
            steps {
                echo 'Running automated tests...'
                // Later, this is where we would run our unit tests.
            }
        }
    }
}
