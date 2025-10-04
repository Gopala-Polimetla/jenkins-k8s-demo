pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'jenkins-k8s-demo'
        DOCKER_TAG = "${BUILD_NUMBER}"
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
                    echo "Built image: ${DOCKER_IMAGE}:${DOCKER_TAG}"
                }
            }
        }
        
        stage('Test Application') {
            steps {
                echo 'Testing application...'
                script {
                    sh """
                        echo "Starting test container..."
                        docker run --rm -d --name test-container-${BUILD_NUMBER} -p 300${BUILD_NUMBER}:3000 ${DOCKER_IMAGE}:${DOCKER_TAG}
                        sleep 15
                        echo "Testing health endpoint..."
                        curl -f http://localhost:300${BUILD_NUMBER}/health || exit 1
                        echo "Test passed!"
                        docker stop test-container-${BUILD_NUMBER}
                    """
                }
            }
        }
        
        stage('Check Kubernetes') {
            steps {
                echo 'Checking Kubernetes cluster...'
                script {
                    sh """
                        kubectl get nodes
                        kubectl get deployments
                        kubectl get services
                    """
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying to Kubernetes...'
                script {
                    sh """
                        echo "Current deployment status:"
                        kubectl get deployment jenkins-k8s-demo || echo "Deployment not found"
                        
                        echo "Updating deployment configuration..."
                        sed -i.bak 's|image: nginx:alpine|image: ${DOCKER_IMAGE}:${DOCKER_TAG}|g' k8s/deployment.yaml
                        sed -i.bak 's|containerPort: 80|containerPort: 3000|g' k8s/deployment.yaml
                        sed -i.bak 's|port: 80|port: 3000|g' k8s/deployment.yaml
                        sed -i.bak 's|targetPort: 80|targetPort: 3000|g' k8s/service.yaml
                        sed -i.bak 's|path: /|path: /health|g' k8s/deployment.yaml
                        sed -i.bak 's|port: 80|port: 3000|g' k8s/deployment.yaml
                        
                        echo "Applying updated configuration..."
                        kubectl apply -f k8s/
                        
                        echo "Waiting for rollout..."
                        kubectl rollout status deployment/jenkins-k8s-demo --timeout=300s
                    """
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                echo 'Verifying deployment...'
                script {
                    sh """
                        echo "Checking pods:"
                        kubectl get pods -l app=jenkins-k8s-demo
                        
                        echo "Checking service:"
                        kubectl get service jenkins-k8s-demo-service
                        
                        echo "Testing application access:"
                        sleep 30
                        curl -f http://103.235.105.210:30080/ || echo "Application not yet ready"
                        curl -f http://103.235.105.210:30080/health || echo "Health check not yet ready"
                    """
                }
            }
        }
    }
    
    post {
        always {
            echo 'Cleaning up...'
            script {
                sh '''
                    docker container prune -f
                    docker image prune -f
                '''
            }
        }
        success {
            echo 'Pipeline completed successfully!'
            echo "Application should be available at: http://103.235.105.210:30080"
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}