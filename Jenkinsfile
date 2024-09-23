pipeline {
    agent any

    environment {
        GITHUB_TOKEN = credentials('jenkin-personal') // Use the ID of your GitHub token credential
        GITHUB_REPO = 'Swapnil011/tomcat' // Your repository
        GITHUB_SHA = '' // Initialize, will set in stages
    }

    stages {
        stage('Validate GitHub Token') {
            when {
                expression { env.CHANGE_ID != null }
            }
            steps {
                script {
                    def response = sh(script: """
                        curl -s -o /dev/null -w "%{http_code}" -H "Authorization: token ${GITHUB_TOKEN}" \
                        "https://api.github.com/user"
                    """, returnStdout: true).trim()
                    
                    if (response != '200') {
                        error "GitHub token validation failed. HTTP Status: ${response}"
                    } else {
                        echo "GitHub token is valid."
                    }
                }
            }
        }

        stage('Preparation') {
            when {
                expression { env.CHANGE_ID != null }
            }
            steps {
                script {
                    echo 'Preparing for the pipeline...'
                    echo "BRANCH NAME: ${env.BRANCH_NAME}"
                    echo sh(returnStdout: true, script: 'env')
                }
            }
        }

        stage('Testing') {
            when {
                expression { env.CHANGE_ID != null }
            }
            steps {
                script {
                    echo 'Running tests...'
                    // Simulate a test command
                    def testResult = sh(script: 'echo "Test passed!"', returnStdout: true).trim()
                    echo "Test Result: ${testResult}"
                }
            }

        }

        stage('Build') {
            when {
                expression { env.CHANGE_ID != null }
            }
            steps {
                script {
                    echo 'Build Started'
                    // Capture the GIT_COMMIT SHA in this stage
                    GITHUB_SHA = env.GIT_COMMIT ?: sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                    echo "Using GITHUB_SHA: ${GITHUB_SHA}"
                    // Simulate a build command
                    sh 'echo "Building application..."'
                }
            }
        }
    }

    post {
        success {
            script {
                if (GITHUB_SHA) {
                    echo "Commit SHA is: ${GITHUB_SHA}" // Ensure correct SHA is set
                    updateGitHubStatus('success')
                } else {
                    echo "No commit SHA available, skipping GitHub status update."
                }
            }
        }
        failure {
            script {
                if (GITHUB_SHA) {
                    echo "Commit SHA is: ${GITHUB_SHA}" // Ensure correct SHA is set
                    updateGitHubStatus('failure')
                } else {
                    echo "No commit SHA available, skipping GitHub status update."
                }
            }
        }
        unstable {
            script {
                if (GITHUB_SHA) {
                    echo "Commit SHA is: ${GITHUB_SHA}" // Ensure correct SHA is set
                    updateGitHubStatus('error')
                } else {
                    echo "No commit SHA available, skipping GitHub status update."
                }
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
