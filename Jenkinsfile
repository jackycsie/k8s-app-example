pipeline {
    agent any
    
    tools {
        nodejs 'NodeJS-18'
    }
    
    environment {
        DOCKER_REGISTRY = 'ghcr.io'
        IMAGE_NAME = "${DOCKER_REGISTRY}/${env.GITHUB_REPOSITORY}"
        DOCKER_CREDENTIALS = credentials('github-token')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm test --if-present'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                        apt-get update
                        apt-get install -y docker.io
                    '''
                    def imageTag = "${env.BUILD_NUMBER}-${env.GIT_COMMIT.take(7)}"
                    def fullImageName = "${IMAGE_NAME}:${imageTag}"
                    
                    sh """
                        docker build -t ${fullImageName} .
                        docker tag ${fullImageName} ${IMAGE_NAME}:latest
                    """
                    
                    // Push to registry
                    sh """
                        echo ${DOCKER_CREDENTIALS_PSW} | docker login ${DOCKER_REGISTRY} -u ${DOCKER_CREDENTIALS_USR} --password-stdin
                        docker push ${fullImageName}
                        docker push ${IMAGE_NAME}:latest
                    """
                }
            }
        }
        
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}