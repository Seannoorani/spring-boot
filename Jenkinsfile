pipeline {
    agent any

    tools {
        // Using the version your error log confirmed exists
        maven 'maven-3.8.5'
        
        // NOTE: You must check 'Manage Jenkins' > 'Tools' to see your exact JDK name.
        // If 'jdk-17.0.17' still fails, try naming it 'null' or checking the tool list.
        jdk 'jdk-17.0.17' 
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
        timestamps()
        // Removed ansiColor to prevent the 'Invalid option' error
    }

    stages {
        stage('Verify Environment') {
            steps {
                sh 'mvn -version'
                sh 'java -version'
            }
        }

        stage('Build') {
            steps {
                // Using -B (Batch mode) is best practice for Jenkins
                sh 'mvn -B clean compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn -B test'
            }
            post {
                always {
                    // This will only work if you have the JUnit plugin installed
                    junit '**/target/surefire-reports/*.xml'
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
            echo 'Build Successful!'
        }
        failure {
            echo 'Build Failed. Please check the logs above.'
        }
    }
}