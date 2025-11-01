pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        STACK_PREFIX = 'NetworkInfraStack'
        AWS_CREDENTIALS = credentials('aws-credentials')  // Jenkins credential ID
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "📦 Checking out code from GitHub..."
                git branch: 'main', url: 'https://github.com/Sanjana1455/EC2-ALB-NLB.git'
            }
        }

        stage('Deploy EC2 Stack') {
            steps {
                echo "🚀 Deploying EC2 Stack..."
                sh """
                aws cloudformation deploy \
                  --template-file templates/ec2.yaml \
                  --stack-name ${STACK_PREFIX}-EC2 \
                  --region ${AWS_REGION} \
                  --capabilities CAPABILITY_NAMED_IAM \
                  --no-fail-on-empty-changeset \
                  --profile default
                """
            }
        }

        stage('Deploy ALB Stack') {
            steps {
                echo "🚀 Deploying ALB Stack..."
                sh """
                aws cloudformation deploy \
                  --template-file templates/alb.yaml \
                  --stack-name ${STACK_PREFIX}-ALB \
                  --region ${AWS_REGION} \
                  --capabilities CAPABILITY_NAMED_IAM \
                  --no-fail-on-empty-changeset \
                  --profile default
                """
            }
        }

        stage('Deploy NLB Stack') {
            steps {
                echo "🚀 Deploying NLB Stack..."
                sh """
                aws cloudformation deploy \
                  --template-file templates/nlb.yaml \
                  --stack-name ${STACK_PREFIX}-NLB \
                  --region ${AWS_REGION} \
                  --capabilities CAPABILITY_NAMED_IAM \
                  --no-fail-on-empty-changeset \
                  --profile default
                """
            }
        }
    }

    post {
        success {
            echo "✅ All CloudFormation stacks deployed successfully!"
        }
        failure {
            echo "❌ Deployment failed. Please check the CloudFormation console."
        }
    }
}
