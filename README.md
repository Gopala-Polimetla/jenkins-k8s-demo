# Jenkins K8s Demo Application

A simple Node.js application for demonstrating CI/CD with Jenkins and Kubernetes.

## Features

- Simple Express.js REST API
- Health check endpoint
- Docker containerization
- Kubernetes deployment manifests
- Jenkins pipeline configuration

## Endpoints

- `GET /` - Main application endpoint
- `GET /health` - Health check endpoint

## Local Development

```bash
# Install dependencies
npm install

# Start application
npm start

# Test health endpoint
curl http://localhost:3000/health
```

## Docker

```bash
# Build image
docker build -t jenkins-k8s-demo .

# Run container
docker run -p 3000:3000 jenkins-k8s-demo
```

## Kubernetes Deployment

```bash
# Apply manifests
kubectl apply -f k8s/

# Check deployment
kubectl get pods,services

# Access application (using NodePort)
curl http://<node-ip>:30080
```

## Jenkins Pipeline

The `Jenkinsfile` contains a complete CI/CD pipeline that:

1. Checks out code from Git
2. Builds Docker image
3. Tests the application
4. Deploys to Kubernetes
5. Verifies the deployment

## Architecture

```
Developer → Git Push → Jenkins → Docker Build → K8s Deploy → Application Running
```