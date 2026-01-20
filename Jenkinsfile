pipeline {
    agent any

    options {
        disableConcurrentBuilds()
        ansiColor('xterm')
        timestamps()
    }

    environment {
        // Ensure these IDs match exactly what is in your Jenkins Credentials store
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key')
        AWS_DEFAULT_REGION    = 'us-east-1' 
    }

    stages {
        stage('Build Lambda Package') {
            steps {
                echo 'Building...'
                sh 'mvn clean install'
            }
        }

        stage('Test') {
            steps {
                echo 'Testing...'
                sh 'mvn test'
            }
        }

        stage('Push to Artifactory') {
            steps {
                echo 'Pushing to artifactory...'
                // Add your upload logic here (e.g., mvn deploy)
            }
        }

        stage('Deploy to QA') {
            steps {
                echo 'Deploying to QA...'
                sh 'zip -g Noorany-0.0.1.zip **/target'
                
                // Using the environment variables directly is cleaner than 'aws configure'
                sh 'aws s3 cp Noorany-0.0.1.zip s3://noorany-lambda-deployments/Noorany-0.0.1.zip'
                sh 'aws lambda update-function-code --function-name test --zip-file fileb://Noorany-0.0.1.zip'
            }
        }

        stage('Release to Prod Approval') {
            // Note: input should usually happen before the actual deployment stage
            steps {
                script {
                    if (env.BRANCH_NAME != 'main') {
                        input message: 'Proceed for Prod Deployment?', ok: 'Deploy'
                    }
                }
            }
        }

        stage('Deploy to Production') {
            steps {
                echo 'Deploying to Production...'
                sh 'aws s3 cp Noorany-0.0.1.zip s3://noorany-lambda-deployments/Noorany-0.0.1.zip'
                sh 'aws lambda update-function-code --function-name prod --zip-file fileb://Noorany-0.0.1.zip'
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed.'
        }
        success {
            echo 'Pipeline completed successfully.'
        }
    }
}