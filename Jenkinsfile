pipeline {
    agent any

    options {
        disableConcurrentBuilds()
        timestamps()
    }

    // Removed the credentials() calls that were causing the error
    environment {
        AWS_DEFAULT_REGION = 'us-east-1' 
    }

    stages {
        stage('Build & Test') {
            steps {
                echo 'Building and Testing...'
                sh 'mvn clean install'
                sh 'mvn test'
            }
        }

        stage('Package') {
            steps {
                echo 'Creating Lambda Package...'
                sh 'zip -g Noorany-0.0.1.zip **/target'
            }
        }

        stage('Deploy to QA') {
            steps {
                echo 'Deploying to QA...'
                // These commands will now rely on the Jenkins Server's IAM Role
                sh 'aws s3 cp Noorany-0.0.1.zip s3://noorany-lambda-deployments/Noorany-0.0.1.zip'
                sh 'aws lambda update-function-code --function-name test --zip-file fileb://Noorany-0.0.1.zip'
            }
        }

        stage('Production Approval') {
            steps {
                script {
                    if (env.BRANCH_NAME != 'main') {
                        input message: 'Proceed to Production?', ok: 'Deploy'
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
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}