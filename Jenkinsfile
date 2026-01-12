pipeline {
    agent any

    tools {
        maven 'Maven-3.8.5'
    }

    stages {
        stage('Package') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Deploy WAR') {
            steps {
                echo 'Deploying WAR...'
                // sh 'scp target/*.war user@server:/deploy/path/'
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: 'target/*.war'
            echo 'Build Success'
        }
        failure {
            echo 'Build Failed'
        }
    }
}
