pipeline {
    agent any
    stages {
        stage('Execute Tests') {
            steps {
                runTests('Unit Tests')
            }
        }
    }
}

// Function to handle test execution and status publishing
def runTests(String testName) {
    script {
        publishChecks(name: testName, status: 'IN_PROGRESS')

        // Simulate test execution with a delay
        sleep(time: 5, unit: 'SECONDS')

        publishChecks(name: testName, status: 'SUCCESS')
    }
}
