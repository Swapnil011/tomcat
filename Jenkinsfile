pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                script {
                    publishChecks(context: 'continuous-integration/jenkins/pr-merge', status: 'pending')

                    // Your build commands here
                    sh 'echo "hello stage build"'

                    publishChecks(context: 'continuous-integration/jenkins/pr-merge', status: 'success')
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    // Your test commands here
                    sh 'echo "hello stage Test"'

                    publishChecks(context: 'continuous-integration/jenkins/pr-merge', status: 'success')
                }
            }
        }
    }
    post {
        failure {
            script {
                publishChecks(context: 'continuous-integration/jenkins/pr-merge', status: 'failure')
            }
        }
    }
}
