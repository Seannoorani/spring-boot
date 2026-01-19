pipeline {
    agent any

    tools {
        // Jenkins global tool configuration
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        AWS_REGION = 'us-east-1'
        LAMBDA_FUNCTION_NAME = 'my-java-lambda'
        MAVEN_OPTS = '-Dmaven.test.failure.ignore=false'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package Lambda') {
            steps {
                // Creates a fat JAR using maven-shade or similar plugin
                sh 'mvn package -DskipTests'
            }
        }

        stage('Deploy to AWS Lambda') {
            when {
                branch 'main'
            }
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws-jenkins-creds']
                ]) {
                    sh '''
                      aws lambda update-function-code \
                        --function-name $LAMBDA_FUNCTION_NAME \
                        --region $AWS_REGION \
                        --zip-file fileb://target/my-lambda.jar
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Lambda deployment successful'
        }
        failure {
            echo '❌ Pipeline failed'
        }
    }
}
