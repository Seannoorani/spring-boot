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
                echo 'Building and running tests'
                sh 'mvn clean verify'
            }
        }

        stage('SonarQube Analysis') {
            when {
                branch 'Buggfix'
            }
            steps {
                echo 'Running SonarQube analysis'
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Archive Artifact') {
            steps {
                echo 'Archiving build artifact'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Push to Artifactory') {
            steps {
                echo 'Pushing to Artifactory'
                // sh 'mvn deploy'
            }
        }

        stage('Deploy to QA') {
            steps {
                echo 'Deploying to QA'
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

