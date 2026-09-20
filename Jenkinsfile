pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test Webhook') {
            steps {
                sh 'echo "GitHub webhook triggered Jenkins successfully!"'
            }
        }
    }
}
