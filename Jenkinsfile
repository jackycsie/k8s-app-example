pipeline {
    agent any
    
    tools {
        nodejs 'NodeJS-18'
    }
    
    environment {
        DOCKER_REGISTRY = 'ghcr.io'
        IMAGE_NAME = 'ghcr.io/jackycsie/k8s-app-example'
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
                        echo ${DOCKER_CREDENTIALS} | docker login ${DOCKER_REGISTRY} -u 'jackycsie' --password-stdin
                        docker push ${fullImageName}
                        docker push ${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('Update GitOps') {
            steps {
                script {
                    def imageTag = "${env.BUILD_NUMBER}-${env.GIT_COMMIT.take(7)}"
                    withCredentials([string(credentialsId: 'github-token', variable: 'GIT_TOKEN')]) {
                        sh """
                            git config --global user.email "jenkins@example.com"
                            git config --global user.name "Jenkins CI"
                            
                            echo "Cloning GitOps repository..."
                            git clone https://github.com/jackycsie/k8s-gitops-config.git
                            cd k8s-gitops-config
                            
                            echo "Updating deployment image to ${IMAGE_NAME}:${imageTag}..."
                            sed -i'' "s|image: .*|image: ${IMAGE_NAME}:${imageTag}|" overlays/production/deployment.yaml
                            
                            echo "Committing and pushing to GitOps repo..."
                            git add .
                            git commit -m "Update image to ${imageTag}" || true
                            git push https://jackycsie:${GIT_TOKEN}@github.com/jackycsie/k8s-gitops-config.git
                        """
                    }
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