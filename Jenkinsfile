pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = "745392035332"
        REGION = "ap-south-1"
        ECR_REPO = "demo-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $ECR_REPO:$IMAGE_TAG .
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $REGION \
                | docker login --username AWS --password-stdin \
                $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com
                '''
            }
        }

        stage('Tag & Push Image') {
            steps {
                sh '''
                docker tag $ECR_REPO:$IMAGE_TAG \
                $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG

                docker push \
                $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                sed -i "s|IMAGE_TAG|$IMAGE_TAG|g" k8s/deployment.yaml
                kubectl apply -f k8s/
                '''
            }
        }
    }
}

