pipeline {
    agent any

    environment {
        GITHUB_TOKEN = credentials('jenkin-personal')
        GITHUB_REPO = 'Swapnil011/tomcat' // your repo
        GITHUB_SHA = env.GIT_COMMIT
    }

    stages {
        stage('Build') {
            steps {
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
