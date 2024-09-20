pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                script {
                    // Report status to GitHub
                    githubNotify context: 'continuous-integration/jenkins/pr-merge', state: 'pending'
                    
                    // Build steps here
                    sh 'Testing completed and ready to build' // Replace with your build command

                    // On success, update status
                    githubNotify context: 'continuous-integration/jenkins/pr-merge', state: 'success'
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    // Testing steps here
                    sh 'make test' // Replace with your test command
                    
                    // On failure, update status
                    githubNotify context: 'continuous-integration/jenkins/pr-merge', state: 'success' // or 'failure' if tests fail
                }
            }
        }
    }
}
