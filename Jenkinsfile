pipeline {
    agent any

    tools {
        jdk 'jdk'
        maven 'maven-3.8.5'
    }

    options {
        timestamps()
        timeout(time: 10, unit: 'MINUTES')
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Versions') {
            steps {
                sh '''
                    java -version
                    mvn -version
                '''
            }
        }

        stage('Build & Test') {
            steps {
                sh '''
                    mvn -B clean test \
                      -DforkCount=1 \
                      -DreuseForks=false \
                      -Dsurefire.printSummary=true
                '''
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package') {
            steps {
                sh 'mvn -B package -DskipTests'
            }
        }
    }

    post {
        success {
            echo '✅ Build succeeded'
        }
        failure {
            echo '❌ Build failed'
        }
        always {
            cleanWs()
        }
    }
}
