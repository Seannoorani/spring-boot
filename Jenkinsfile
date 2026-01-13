pipeline {
    agent any

    tools {
        jdk 'jdk-17.0.17'
        maven 'maven-3.8.7'
    }

    environment {
        // Define variables for reuse
        DOCKER_HUB_USER = 'your-dockerhub-username'
        APP_NAME        = 'java-service'
        IMAGE_TAG       = "${env.APP_NAME}:${env.BUILD_NUMBER}"
        SCAN_CRITICAL_FAIL = 'true' 
    }

    options {
        timeout(time: 1, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '20'))
        timestamps()
        ansiColor('xterm') // Requires AnsiColor plugin for pretty logs
    }

    stages {
        stage('🚀 Setup & Lint') {
            steps {
                echo "Starting build ${env.BUILD_ID}..."
                sh 'mvn -version'
            }
        }

        stage('🧪 Build & Unit Tests') {
            steps {
                // -T 1C means 1 thread per CPU core for faster parallel builds
                sh 'mvn -B -T 1C clean install -DskipTests=false'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('🛡️ Security Scan (OWASP)') {
            steps {
                echo "Checking dependencies for vulnerabilities..."
                // Dependency-Check identifies project dependencies and checks if there are any known, publicly disclosed vulnerabilities
                sh 'mvn org.owasp:dependency-check-maven:check'
            }
            post {
                always {
                    dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
                }
            }
        }

        stage('📊 Static Analysis (SonarQube)') {
            steps {
                // This requires a SonarQube server configured in Jenkins
                withSonarQubeEnv('SonarQube-Server') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('🐳 Docker Build & Push') {
            when { branch 'main' } // Only push to registry from the main branch
            steps {
                script {
                    dockerImage = docker.build("${IMAGE_TAG}")
                    
                    docker.withRegistry('', 'docker-hub-credentials-id') {
                        dockerImage.push()
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('🚢 Deploy to Staging') {
            when { branch 'main' }
            steps {
                input message: 'Deploy to Staging?', ok: 'Yes, Deploy!'
                echo "Deploying ${IMAGE_TAG} to Staging Environment..."
                // Example: sh 'kubectl set image deployment/my-app my-app=${IMAGE_TAG}'
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
            // You could add Slack notification here
        }
        failure {
            echo "❌ Pipeline failed. Sending alerts..."
            mail to: 'dev-team@example.com',
                 subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
                 body: "Something went wrong with build ${env.BUILD_URL}"
        }
    }
}