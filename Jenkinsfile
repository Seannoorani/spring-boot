pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                // This assumes 'mvn' is already in the system PATH of your Jenkins agent
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
    }

    post {
        always {
            // This will attempt to archive your jar if the build succeeded
            archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
        }
    }
}