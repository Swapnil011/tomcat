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

def runTests(String testName) {
    script {
        publishChecks(name: testName, status: 'NONE')

        // Simulate test execution with a delay
        sleep(time: 5, unit: 'SECONDS')

        // Use a valid status
        publishChecks(name: testName, status: 'SUCCESS') // or any other valid status
    }
}
