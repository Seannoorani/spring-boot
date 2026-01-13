pipeline {
    agent any

    tools {
        // These names MUST match the "Name" field in Global Tool Configuration
        jdk 'jdk-17.0.17'
        maven 'maven-3.8.7'
    }

    options {
        // Keeps the console output clean and limits the number of old builds
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
    }

    stages {
        stage('Environment Check') {
            steps {
                // Verifies that the correct Java and Maven versions are active
                sh 'java -version'
                sh 'mvn -version'
            }
        }

        stage('Clean & Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Run Tests') {
            steps {
                // -B runs Maven in non-interactive (batch) mode to avoid messy logs
                sh 'mvn -B test'
            }
            post {
                always {
                    // Publishes test results so they appear in the Jenkins UI
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package') {
            steps {
                // Create the JAR file, skipping tests since they just passed
                sh 'mvn -B package -DskipTests'
            }
        }

        stage('Archive') {
            steps {
                // Save the built JAR so it can be downloaded from the Jenkins build page
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }

    post {
        failure {
            echo "Build failed. Verify Java 17 compatibility or check test logs."
        }
        success {
            echo "Build and Packaging completed successfully with Java 17.0.17!"
        }
    }
}