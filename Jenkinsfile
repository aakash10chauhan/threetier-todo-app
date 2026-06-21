pipeline {
    agent any

    environment {
        CLUSTER_NAME    = "your-eks-cluster-name" // <-- Change this to your actual EKS cluster name
        NAMESPACE       = "three-tier" // <-- Change this to your actual namespace if different
        APP_VERSION     = "${BUILD_NUMBER}"
        PROD_ACCOUNT_ID = "252610334383"
        AWS_REGION      = "us-east-1"
        ECR_BASE        = "${PROD_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        FRONT_REPO      = "${ECR_BASE}/todo-frontend"
        BACK_REPO       = "${ECR_BASE}/todo-backend"
        IMAGE_TAG       = "v1.0.${BUILD_NUMBER}"
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/aakash10chauhan/threetier-todo-app.git'
            }
        }

        stage('Build Docker images') {
            steps {
                echo "building frontend..."
                sh "docker build -t frontend:latest Application-Code/frontend"

                echo "building backend..."
                sh "docker build -t backend:latest Application-Code/backend"
            }
        }

        stage('Login to ECR') {
            steps {
                sh "aws ecr get-login-password --region ${AWS_REGION} \
                      | docker login --username AWS --password-stdin ${ECR_BASE}"
            }
        }

        stage('tag images to latest') {
            steps {
                echo "Tagging frontend image..."
                sh "docker tag frontend:latest ${FRONT_REPO}:${IMAGE_TAG}"

                echo "Tagging backend image..."
                sh "docker tag backend:latest ${BACK_REPO}:${IMAGE_TAG}"
            }
        }
        
        stage('Push latest tag to ECR') {
            steps {
                echo "Pushing frontend image to ECR..."
                sh "docker push ${FRONT_REPO}:${IMAGE_TAG}"
                
                echo "Pushing backend image to ECR..."
                sh "docker push ${BACK_REPO}:${IMAGE_TAG}"
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    echo "Connecting to EKS Cluster..."
                    // 1. Authenticate kubectl with your EKS cluster
                    sh "aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}"
                    
                    echo "Updating EKS deployments with image tag ${IMAGE_TAG}..."
                    
                    // 2. Update the deployment images (Update deployment and container names to match your manifest)
                    // Syntax: kubectl set image deployment/<DEPLOYMENT_NAME> <CONTAINER_NAME>=<IMAGE> -n <NAMESPACE>
                    sh "kubectl set image deployment/frontend-deployment frontend-container=${FRONT_REPO}:${IMAGE_TAG} -n ${NAMESPACE} --record"
                    sh "kubectl set image deployment/backend-deployment backend-container=${BACK_REPO}:${IMAGE_TAG} -n ${NAMESPACE} --record"

                    echo "Verifying deployment rollout status..."
                    // 3. Optional: Verify that the rollout succeeds
                    sh "kubectl rollout status deployment/frontend-deployment -n ${NAMESPACE}"
                    sh "kubectl rollout status deployment/backend-deployment -n ${NAMESPACE}"
                }
            }
        }
    }

    post {
        success {
            echo "Todo app version ${IMAGE_TAG} deployed to EKS successfully"
        }
        failure {
            echo "Deployment failed for version ${IMAGE_TAG}"
        }
    }
}