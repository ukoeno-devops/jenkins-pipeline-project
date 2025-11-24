pipeline {
    agent any

    environment {
        AWS_REGION      = 'us-east-1'
        AWS_ACCOUNT_ID  = '157187738117'
        ECR_REPO        = 'myapp-repo'
        IMAGE_TAG       = "latest"
        ECR_URI         = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
        CLUSTER_NAME    = 'my-ecs-cluster'
        SERVICE_NAME    = 'myapp-service'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ukoeno-devops/jenkins-pipeline-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t \$ECR_REPO:\$IMAGE_TAG .
                """
            }
        }

        stage('Login to ECR') {
            steps {
                sh """
                aws ecr get-login-password --region \$AWS_REGION | docker login --username AWS --password-stdin \$AWS_ACCOUNT_ID.dkr.ecr.\$AWS_REGION.amazonaws.com
                """
            }
        }

        stage('Tag & Push Image') {
            steps {
                sh """
                docker tag \$ECR_REPO:\$IMAGE_TAG \$ECR_URI:\$IMAGE_TAG
                docker push \$ECR_URI:\$IMAGE_TAG
                """
            }
        }

        stage('Deploy to ECS') {
            steps {
                sh """
                aws ecs update-service \
                  --cluster \$CLUSTER_NAME \
                  --service \$SERVICE_NAME \
                  --force-new-deployment \
                  --region \$AWS_REGION
                """
            }
        }
    }

    post {
        success {
            echo '✅ Deployment successful!'
        }
        failure {
            echo '❌ Deployment failed.'
        }
    }
}

