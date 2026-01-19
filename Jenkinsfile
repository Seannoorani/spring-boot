pipeline {
    agent {
        docker {
            image 'maven:3.9.6-eclipse-temurin-17'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        AWS_REGION = 'us-east-1'
        LAMBDA_NAME = 'my-java-lambda'
        JAR_FILE = 'lambda.jar'
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {

        /* =====================
           CI
           ===================== */

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        /* =====================
           CD
           ===================== */

        stage('Deploy to Lambda') {
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
                          --function-name $LAMBDA_NAME \
                          --region $AWS_REGION \
                          --zip-file fileb://target/$JAR_FILE
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ CI/CD pipeline completed successfully'
        }
        failure {
            echo '❌ Pipeline failed'
        }
        always {
            cleanWs()
        }
    }
}

