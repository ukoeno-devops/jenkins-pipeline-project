pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '157187738117'
        ECR_REPO = 'myapp-repo'
        IMAGE_TAG = "latest"
        ECR_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ukoeno-devops/jenkins-pipeline-project.git/'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t $ECR_REPO:$IMAGE_TAG .'
                }
            }
        }

        stage('Login to ECR') {
            steps {
                script {
                    sh 'aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_URI'
                }
            }
        }

        stage('Tag and Push Image') {
            steps {
                script {
                    sh '''
                        docker tag $ECR_REPO:$IMAGE_TAG $ECR_URI:$IMAGE_TAG
                        docker push $ECR_URI:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy to ECS') {
            steps {
                script {
                    sh '''
                        aws ecs update-service \
                            --cluster your-cluster-name \
                            --service your-service-name \
                            --force-new-deployment \
                            --region $AWS_REGION
                    '''
                }
            }
        }
    }
}

