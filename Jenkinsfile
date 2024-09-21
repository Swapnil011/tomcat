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
                echo "Commit SHA is: ${GITHUB_SHA}" // Ensure correct SHA is set
                updateGitHubStatus('success')
            }
        }
        failure {
            script {
                echo "Commit SHA is: ${GITHUB_SHA}" // Ensure correct SHA is set
                updateGitHubStatus('failure')
            }
        }
        unstable {
            script {
                echo "Commit SHA is: ${GITHUB_SHA}" // Ensure correct SHA is set
                updateGitHubStatus('error')
            }
        }
    }
}

def updateGitHubStatus(String status) {
    echo "Updating GitHub status to '${status}' for commit SHA '${GITHUB_SHA}'"
    sh """
        curl -X POST -H "Authorization: token ${GITHUB_TOKEN}" \
        -d '{"state": "${status}", "context": "continuous-integration/jenkins"}' \
        "https://api.github.com/repos/${GITHUB_REPO}/statuses/${GITHUB_SHA}"
    """
}
