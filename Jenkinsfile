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
        jdk 'JDK-17'
        maven 'Maven-3.9'
    }

    /***********************
     * PARAMETERS
     ***********************/
    parameters {
        string(name: 'GIT_BRANCH', defaultValue: 'main', description: 'Git branch to build')
        choice(name: 'ENVIRONMENT', choices: ['dev', 'qa', 'prod'], description: 'Deployment environment')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run unit tests')
        booleanParam(name: 'DEPLOY', defaultValue: false, description: 'Deploy after build')
        booleanParam(name: 'BUILD_DOCKER', defaultValue: false, description: 'Build Docker image')
    }

    /***********************
     * ENVIRONMENT VARIABLES
     ***********************/
    environment {
        APP_NAME = 'my-app'
        MAVEN_OPTS = '-Xmx1024m'
        DEPLOY_PATH = '/opt/apps'
        DOCKER_IMAGE = "myorg/my-app:${BUILD_NUMBER}"
        SSH_CREDENTIALS = 'ssh-server-key'   // Jenkins credential ID
    }

    /***********************
     * STAGES
     ***********************/
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
                sh 'mvn clean compile'
            }
        }

        stage('Test') {
            when {
                expression { params.RUN_TESTS }
            }
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.war', fingerprint: true
            }
        }

        stage('Build Docker Image') {
            when {
                expression { params.BUILD_DOCKER }
            }
            steps {
                sh '''
                docker build -t ${DOCKER_IMAGE} .
                docker images | grep my-app
                '''
            }
        }

        stage('Deploy') {
            when {
                allOf {
                    expression { params.DEPLOY }
                    expression { params.ENVIRONMENT != 'prod' || currentBuild.currentResult == 'SUCCESS' }
                }
            }
            steps {
                echo "Deploying to ${params.ENVIRONMENT}"

                sshagent(credentials: [env.SSH_CREDENTIALS]) {
                    sh """
                    scp target/*.war user@server:${DEPLOY_PATH}/${APP_NAME}.war
                    ssh user@server 'systemctl restart ${APP_NAME}'
                    """
                }
            }
        }
    }

    /***********************
     * POST ACTIONS
     ***********************/
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
            cleanWs()
        }
    }
}