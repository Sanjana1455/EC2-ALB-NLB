pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        STACK_PREFIX = 'NetworkInfraStack'
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
    withAWS(region: "${AWS_REGION}", credentials: 'aws-cred') {
      sh '''
        aws cloudformation deploy \
          --template-file templates/ec2-template.yaml \
          --stack-name NetworkInfraStack-EC2 \
          --region us-east-1 \
          --capabilities CAPABILITY_NAMED_IAM \
          --no-fail-on-empty-changeset
      '''
    }
  }
}


        stage('Deploy ALB Stack') {
            steps {
                echo "🌐 Deploying ALB Stack..."
                withAWS(credentials: 'aws-cred', region: "${AWS_REGION}") {
                    sh """
                    aws cloudformation deploy \
                      --template-file templates/alb-template.yaml \
                      --stack-name ${STACK_PREFIX}-ALB \
                      --region ${AWS_REGION} \
                      --capabilities CAPABILITY_NAMED_IAM \
                      --no-fail-on-empty-changeset
                    """
                }
            }
        }

        stage('Deploy NLB Stack') {
            steps {
                echo "🔁 Deploying NLB Stack..."
                withAWS(credentials: 'aws-cred', region: "${AWS_REGION}") {
                    sh """
                    aws cloudformation deploy \
                      --template-file templates/nlb-template.yaml \
                      --stack-name ${STACK_PREFIX}-NLB \
                      --region ${AWS_REGION} \
                      --capabilities CAPABILITY_NAMED_IAM \
                      --no-fail-on-empty-changeset
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ All stacks deployed successfully!"
        }
        failure {
            echo "❌ Deployment failed. Please check the CloudFormation console."
        }
    }
}
