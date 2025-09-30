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
            when {
                branch 'main'
            }
            steps {
                script {
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
        
        stage('Update GitOps') {
            when {
                branch 'main'
            }
            steps {
                script {
                    def imageTag = "${env.BUILD_NUMBER}-${env.GIT_COMMIT.take(7)}"
                    
                    // Clone GitOps repository
                    sh """
                        git clone https://github.com/${env.GITHUB_REPOSITORY_OWNER}/k8s-gitops-config.git
                        cd k8s-gitops-config
                        
                        # Update deployment image
                        sed -i 's|image: .*|image: ${IMAGE_NAME}:${imageTag}|' overlays/production/deployment.yaml
                        
                        git config user.name "Jenkins"
                        git config user.email "jenkins@example.com"
                        git add .
                        git commit -m "Update image to ${imageTag}" || true
                        git push https://${DOCKER_CREDENTIALS_USR}:${DOCKER_CREDENTIALS_PSW}@github.com/${env.GITHUB_REPOSITORY_OWNER}/k8s-gitops-config.git
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