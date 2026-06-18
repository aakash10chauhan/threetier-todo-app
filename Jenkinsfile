pipeline {
    agent any

    environment {
        APP_VERSION     = "${BUILD_NUMBER}"
        PROD_ACCOUNT_ID = "252610334383"
        AWS_REGION      = "us-east-1"
        ECR_BASE        = "${PROD_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        FRONT_REPO     = "${ECR_BASE}/todo-frontend"
        BACK_REPO      = "${ECR_BASE}/todo-backend"
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
                echo "Tagging frontend image to latest..."
                sh "docker tag frontend:latest ${FRONT_REPO}:${IMAGE_TAG}"

                echo "Tagging backend image to latest..."
                sh "docker tag backend:latest ${BACK_REPO}:${IMAGE_TAG}"
            }
        }
        
        stage('Push latest tag to ECR') {
            steps {
                echo "Pushing frontend latest image to ECR..."
                sh "docker push ${FRONT_REPO}:${IMAGE_TAG}"
                
                echo "Pushing backend latest image to ECR..."
                sh "docker push ${BACK_REPO}:${IMAGE_TAG}"
            }
        }
    }

    post {
        success {
            echo "Todo app version ${IMAGE_TAG} deployed to ECR successfully"
        }
        failure {
            echo "Deployment failed for version ${IMAGE_TAG}"
        }
    }
}