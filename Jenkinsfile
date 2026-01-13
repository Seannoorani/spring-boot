pipeline {
    agent any

    tools {
        // We keep Maven because your log suggested 'maven-3.8.5' works
        maven 'maven-3.8.5'
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
        timestamps()
    }

    stages {
        stage('Check Versions') {
            steps {
                // This will show us what version is actually on the machine
                sh 'java -version'
                sh 'mvn -version'
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn -B clean install'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
    }
}