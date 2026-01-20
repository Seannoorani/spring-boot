pipeline {
    agent any

    parameters {
        booleanParam(name: 'SKIP_TESTS', defaultValue: false, description: 'Check to skip Maven tests')
    }

    environment {
        AWS_REGION  = 'us-east-1'
        LAMBDA_NAME = 'my-java-lambda'
        // Updated to match your Maven log: [Building hello-java 0.1.0]
        JAR_FILE    = 'hello-java-0.1.0.jar'
        AWS_CREDS   = credentials('aws-jenkins-creds')
    }

    options {
        timestamps()
        disableConcurrentBuilds()
        timeout(time: 1, unit: 'HOURS') // Prevents the "infinite hang" you saw
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                // If SKIP_TESTS is true, we pass the skip flag to Maven
                script {
                    def skipFlag = params.SKIP_TESTS ? '-DskipTests' : ''
                    sh "mvn clean package ${skipFlag}"
                }
            }
        }

        stage('Verify Artifact') {
            steps {
                // Defensive check: ensure the file actually exists before trying to deploy
                sh "ls -lh target/${JAR_FILE}"
            }
        }

        stage('Deploy') {
            when {
                anyOf {
                    branch 'main'
                    branch 'Buggfix' // Added this so you can test deployment now
                }
            }
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', 
                                  credentialsId: 'aws-jenkins-creds']]) {
                    echo "Deploying ${JAR_FILE} to Lambda: ${LAMBDA_NAME}..."
                    sh '''
                        aws lambda update-function-code \
                            --function-name ${LAMBDA_NAME} \
                            --region ${AWS_REGION} \
                            --zip-file fileb://target/${JAR_FILE}
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the console output for errors.'
        }
        always {
            cleanWs()
        }
    }
}