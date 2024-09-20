pipeline {
    agent any
    
    stages {
        stage('Preparation') {
            steps {
                echo 'Preparing for the pipeline...'
                echo "BRANCH NAME: ${env.BRANCH_NAME}"
                echo sh(returnStdout: true, script: 'env')
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
            when {
                branch 'main'
            }
            steps {
                script {
                    echo 'Build Started'
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
            setBuildStatus("Build succeeded", "SUCCESS")
        }
        failure {
            setBuildStatus("Build failed", "FAILURE")
        } 
    }
}

void setBuildStatus(String message, String state) {
    step([
        $class: "GitHubCommitStatusSetter",
        reposSource: [$class: "ManuallyEnteredRepositorySource", url: "https://github.com/Swapnil011/tomcat.git"],
        contextSource: [$class: "ManuallyEnteredCommitContextSource", context: "ci/jenkins/build-status"],
        errorHandlers: [[$class: "ChangingBuildStatusErrorHandler", result: "UNSTABLE"]],
        statusResultSource: [$class: "ConditionalStatusResultSource", results: [[$class: "AnyBuildResult", message: message, state: state]]]
    ])
}
