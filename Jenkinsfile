pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'jenkins-k8s-demo'
        DOCKER_TAG = "${BUILD_NUMBER}"
        KUBECONFIG_CREDENTIAL_ID = 'kubeconfig'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code...'
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                    sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
                }
            }
        }
        
        stage('Test Application') {
            steps {
                echo 'Testing application...'
                script {
                    // Run a quick test container
                    sh """
                        docker run --rm -d --name test-container -p 3001:3000 ${DOCKER_IMAGE}:${DOCKER_TAG}
                        sleep 10
                        curl -f http://localhost:3001/health || exit 1
                        docker stop test-container
                    """
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying to Kubernetes...'
                script {
                    // Update deployment with new image
                    sh """
                        kubectl set image deployment/jenkins-k8s-demo jenkins-k8s-demo=${DOCKER_IMAGE}:${DOCKER_TAG}
                        kubectl rollout status deployment/jenkins-k8s-demo
                    """
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                echo 'Verifying deployment...'
                script {
                    sh """
                        kubectl get pods -l app=jenkins-k8s-demo
                        kubectl get services jenkins-k8s-demo-service
                    """
                }
            }
        }
    }
    
    post {
        always {
            echo 'Cleaning up...'
            sh 'docker system prune -f'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}