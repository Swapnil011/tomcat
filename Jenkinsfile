pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                script {
                    publishChecks(name: 'Test', status: 'IN_PROGRESS')
                    // Simulate build or test
                    sleep(time: 5, unit: 'SECONDS')
                    publishChecks(name: 'Test', status: 'SUCCESS')
                }
            }
        }
    }
}
