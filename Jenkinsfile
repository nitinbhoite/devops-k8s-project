pipeline {
    agent any
    environment {
        PATH = "/usr/bin:${env.PATH}"
        AWS_REGION = 'ap-south-1'
        ECR_REPOSITORY = 'devops-k8s-demo'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                echo 'Building application...'
            }
        }
        stage('Test') {
            steps {
                echo 'Running tests...'
            }
        }
        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                    -t ${ECR_REPOSITORY}:${IMAGE_TAG} .
                '''
            }
        }
        stage('ECR Login') {
            steps {
                sh '''
                    AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

                    echo "AWS Account: $AWS_ACCOUNT_ID"
                    echo "AWS Region: ${AWS_REGION}"

                    aws ecr get-login-password --region ${AWS_REGION} |
                    docker login --username AWS \
                    --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                '''
            }
        }
        stage('Docker Push') {
            steps {
                sh '''
                    AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

                    ECR_URI=${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}

                    echo "ECR URI: ${ECR_URI}"

                    docker tag devops-k8s-demo:${BUILD_NUMBER} ${ECR_URI}:${BUILD_NUMBER}

                    docker push ${ECR_URI}:${BUILD_NUMBER}
                '''
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                '''
            }
        }
        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl rollout status deployment/devops-demo
                '''
            }
        }
    }
}