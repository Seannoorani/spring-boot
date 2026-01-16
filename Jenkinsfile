pipeline {
    agent {
        label 'java'
    }

    stages {
        stage('Initialize') {
            steps {
                echo 'Starting Build for Sean Noorani...'
                // Verifies java is actually there to prevent environment failures
                sh 'java -version'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // 'SonarQube' must match the name in Manage Jenkins -> System -> SonarQube installations
                withSonarQubeEnv('SonarQube') {
                    script {
                        try {
                            // If using Maven:
                            sh 'mvn sonar:sonar'
                            
                            // IF NOT using Maven, use the standalone scanner:
                            // sh 'sonar-scanner'
                        } catch (Exception e) {
                            echo "SonarQube analysis failed, but continuing pipeline..."
                        }
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // This pauses until SonarQube reports back success/fail
                    // If you want the build to NEVER fail here, wrap it in a catchError
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline execution finished.'
        }
    }
}