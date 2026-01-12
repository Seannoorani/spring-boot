pipeline {

    /***********************
     * AGENT
     ***********************/
    agent any

    /***********************
     * GLOBAL OPTIONS
     ***********************/
    options {
        buildDiscarder(logRotator(
            numToKeepStr: '10',
            artifactNumToKeepStr: '5'
        ))
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        timestamps()
    }

    /***********************
     * TOOLS
     ***********************/
    tools {
        // Updated to match the name Jenkins suggested in your error log
        maven 'maven-3.8.5'
        // If 'JDK-17' failed, you may need to check your Global Tool Configuration 
        // for the exact name, or use 'Default' if one isn't named.
        jdk 'jdk-17' 
    }

    stages {

        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "*/${params.GIT_BRANCH}"]],
                    userRemoteConfigs: [[
                        url: 'https://github.com/your-org/your-repo.git'
                    ]]
                ])
            }
        }

        stage('Build') {
            steps {
                // Use 'sh' for Linux/Mac or 'bat' for Windows
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            when {
                expression { params.RUN_TESTS }
            }
            steps {
                sh 'mvn test'
            }
        }
    }

    post {
        success {
            echo '✅ Build & Pipeline Successful'
        }

        failure {
            echo '❌ Build Failed'
        }

        unstable {
            echo '⚠️ Build Unstable'
        }

        always {
            archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            junit '**/target/surefire-reports/*.xml'
        }
    }
}