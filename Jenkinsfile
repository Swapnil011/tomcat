pipeline {
    agent any

    environment {
        GITHUB_TOKEN = credentials('jenkin-personal') // Use the ID of your GitHub token credential
        GITHUB_REPO = 'Swapnil011/tomcat' // Your repository
        GITHUB_SHA = '' // Initialize, will set in stages
    }

    stages {
        stage('Build') {
            steps {
                script {
                    // Capture the GIT_COMMIT SHA in this stage
                    GITHUB_SHA = env.GIT_COMMIT
                }
                echo 'Building...'
                // Add your build steps here
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
                // Add your test steps here
            }
        }
    }

    post {
        success {
            script {
                updateGitHubStatus('success')
            }
        }
        failure {
            script {
                updateGitHubStatus('failure')
            }
        }
        unstable {
            script {
                updateGitHubStatus('error')
            }
        }
    }
}

def updateGitHubStatus(String status) {
    sh """
        curl -X POST -H "Authorization: token ${GITHUB_TOKEN}" \
        -d '{"state": "${status}", "context": "continuous-integration/jenkins"}' \
        "https://api.github.com/repos/${GITHUB_REPO}/statuses/${GITHUB_SHA}"
    """
}
