pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        LAMBDA_NAME = 'my-java-lambda'
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {

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

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws-jenkins-creds']
                ]) {
                    sh '''
                        JAR_FILE=$(ls target/*.jar | grep -v original | head -n 1)

                        echo "Deploying $JAR_FILE"

                        aws lambda update-function-code \
                          --function-name $LAMBDA_NAME \
                          --region $AWS_REGION \
                          --zip-file fileb://$JAR_FILE
                    '''
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
