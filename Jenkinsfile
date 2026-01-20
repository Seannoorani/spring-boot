pipeline {
    agent any

    options {
        disableConcurrentBuilds()
        ansiColor('xterm')
        timestamps()
    }

    environment {
        // AWS CLI uses these variable names automatically for authentication
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key')
        AWS_DEFAULT_REGION    = 'us-east-1' 
    }

    stages {
        stage('Build & Test') {
            steps {
                echo 'Building and Testing with System Maven...'
                // Using 'mvn' directly assumes it is installed on the Jenkins server path
                sh 'mvn clean install'
                sh 'mvn test'
            }
        }

        stage('Package') {
            steps {
                echo 'Creating Lambda Package...'
                // This creates the zip file for deployment
                sh 'zip -g Noorany-0.0.1.zip **/target'
            }
        }

        stage('Deploy to QA') {
            steps {
                echo 'Deploying to QA environment...'
                sh 'aws s3 cp Noorany-0.0.1.zip s3://noorany-lambda-deployments/Noorany-0.0.1.zip'
                sh 'aws lambda update-function-code --function-name test --zip-file fileb://Noorany-0.0.1.zip'
            }
        }

        stage('Production Approval') {
            steps {
                script {
                    // Only ask for permission if we aren't on the main branch
                    // (Or swap to 'if (env.BRANCH_NAME == "main")' if you only want prod from main)
                    if (env.BRANCH_NAME != 'main') {
                        input message: 'Proceed to Production?', ok: 'Deploy Now'
                    }
                }
            }
        }

        stage('Deploy to Production') {
            steps {
                echo 'Deploying to Production environment...'
                sh 'aws s3 cp Noorany-0.0.1.zip s3://noorany-lambda-deployments/Noorany-0.0.1.zip'
                sh 'aws lambda update-function-code --function-name prod --zip-file fileb://Noorany-0.0.1.zip'
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
        }
        success {
            echo 'Deployment complete!'
        }
        failure {
            echo 'Build failed. Please check the console output above.'
        }
    }
}