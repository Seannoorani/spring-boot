pipeline {
    agent any

    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        timestamps()
    }

    stages {

        stage('Build & Test') {
            steps {
                echo 'Building application and running tests'
                sh 'mvn clean verify'
            }
        }

        stage('Archive Artifact') {
            steps {
                echo 'Archiving JAR artifact'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Push to Artifactory') {
            steps {
                echo 'Pushing artifact to Artifactory'
                // sh 'mvn deploy'
            }
        }

        stage('Deploy to QA') {
            steps {
                echo 'Deploying application to QA'
                // sh './deploy-qa.sh'
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

        always {
            cleanWs()
        }
    }
}
