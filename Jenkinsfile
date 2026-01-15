pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Build'
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                echo 'Testing...'
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'SonarQube'
            }
        }

        stage('Push to Artifactory') {
            steps {
                echo 'Pushing to Artifactory'
            }
        }

        stage('Deploy to QA') {
            steps {
                echo 'Deploy to QA'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
