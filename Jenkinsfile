pipeline {
    agent any

    environment {
        GITHUB_REPO = 'Swapnil011/tomcat' // Your repository
        GITHUB_SHA = '' // Initialize, will set in stages
    }

    stages {
        stage('Validate GitHub Token') {
            steps {
                withCredentials([string(credentialsId: 'jenkin-personal1', variable: 'GITHUB_TOKEN')]) {
                    script {
                        def response = sh(script: """
                            curl -s -o /dev/null -w "%{http_code}" -H "Authorization: token \$GITHUB_TOKEN" \
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
        }

        stage('Preparation') {
            steps {
                script {
                    echo 'Preparing for the pipeline...'
                    echo "BRANCH NAME: ${env.BRANCH_NAME}"
                    echo sh(returnStdout: true, script: 'env')
                }
            }
        }

        stage('Testing') {
            steps {
                script {
                    echo 'Running tests...'
                    def testResult = sh(script: 'echo "Test passed!"', returnStdout: true).trim()
                    echo "Test Result: ${testResult}"
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    echo 'Build Started'
                    GITHUB_SHA = env.GIT_COMMIT ?: sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                    echo "Using GITHUB_SHA: ${GITHUB_SHA}"
                    sh 'echo "Building application..."'
                }
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                script {
                    echo 'Deploying Application'
                    sh 'echo "Application deployed!"'
                }
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
    withCredentials([string(credentialsId: 'jenkin-personal1', variable: 'GITHUB_TOKEN')]) {
        try {
            echo "Updating GitHub status to '${status}' for commit SHA '${GITHUB_SHA}'"

            def jsonData = [
                state: status,
                context: "continuous-integration/jenkins"
            ]
            def jsonStr = new groovy.json.JsonBuilder(jsonData).toPrettyString()

            def response = sh(script: """
                curl -X POST -H "Accept: application/vnd.github+json" \
                -H "Authorization: Bearer \$GITHUB_TOKEN" \
                -H "X-GitHub-Api-Version: 2022-11-28" \
                -H "Content-Type: application/json; charset=utf-8" \
                "https://api.github.com/repos/${GITHUB_REPO}/statuses/${GITHUB_SHA}" \
                -d '${jsonStr}'
            """, returnStdout: true).trim()

            echo "GitHub response: ${response}"

            def responseJson = new groovy.json.JsonSlurper().parseText(response)
            if (responseJson.state != status) {
                error "Unexpected response from GitHub API: ${responseJson}"
            }
        } catch (Exception e) {
            error "Failed to update GitHub status: ${e.message}"
        }
    }
}
