pipeline {
    agent any

    stages {
        stage('Preparation') {
            steps {
                echo 'Preparing for tests...'
            }
        }

        stage('Testing') {
            steps {
                script {
                    echo 'Running tests...'
                    // Simulate a test command
                    def testResult = sh(script: 'echo "Test passed!"', returnStdout: true).trim()
                    echo "Test Result: ${testResult}"
                }
            }
        }
    }

    post {
        success {
            echo 'All tests passed successfully!'
        }
        failure {
            echo 'Some tests failed.'
        }
    }
}
