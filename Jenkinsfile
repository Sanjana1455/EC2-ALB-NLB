pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        STACK_PREFIX = "NetworkInfraStack"
        KEY_NAME = "sanjana-key"
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "📦 Checking out code from GitHub..."
                git branch: 'main', url: 'https://github.com/Sanjana1455/EC2-ALB-NLB.git'
            }
        }

        stage('Clean Old Stacks') {
            steps {
                echo "🧹 Deleting old CloudFormation stacks if they exist..."
                withAWS(region: "${AWS_REGION}", credentials: 'aws-cred') {
                    sh '''
                    for STACK in EC2 ALB NLB; do
                        aws cloudformation delete-stack --stack-name ${STACK_PREFIX}-$STACK --region ${AWS_REGION} || true
                        aws cloudformation wait stack-delete-complete --stack-name ${STACK_PREFIX}-$STACK --region ${AWS_REGION} || true
                    done
                    echo "✅ Old stacks cleaned up."
                    '''
                }
            }
        }

        stage('Create Key Pair') {
            steps {
                echo "🔐 Checking if key pair exists..."
                withAWS(region: "${AWS_REGION}", credentials: 'aws-cred') {
                    sh '''
                    if ! aws ec2 describe-key-pairs --key-names ${KEY_NAME} --region ${AWS_REGION} >/dev/null 2>&1; then
                        echo "🔑 Key pair not found. Creating new one..."
                        aws ec2 create-key-pair --key-name ${KEY_NAME} --query "KeyMaterial" --output text > ${KEY_NAME}.pem
                        chmod 400 ${KEY_NAME}.pem
                        echo "✅ Key pair '${KEY_NAME}' created and saved as ${KEY_NAME}.pem"
                    else
                        echo "✅ Key pair '${KEY_NAME}' already exists."
                    fi
                    '''
                }
            }
        }

        stage('Deploy EC2 Stack') {
            steps {
                echo "🚀 Deploying EC2 Stack..."
                withAWS(region: "${AWS_REGION}", credentials: 'aws-cred') {
                    sh '''
                    aws cloudformation deploy \
                        --template-file templates/ec2-template.yaml \
                        --stack-name ${STACK_PREFIX}-EC2 \
                        --region ${AWS_REGION} \
                        --parameter-overrides KeyName=${KEY_NAME} \
                        --capabilities CAPABILITY_NAMED_IAM \
                        --no-fail-on-empty-changeset
                    '''
                }
            }
        }

        stage('Deploy ALB Stack') {
            steps {
                echo "🌐 Deploying ALB Stack..."
                withAWS(region: "${AWS_REGION}", credentials: 'aws-cred') {
                    sh '''
                    VPC_ID=$(aws ec2 describe-vpcs --query "Vpcs[0].VpcId" --output text --region ${AWS_REGION})
                    SUBNETS=$(aws ec2 describe-subnets --query "Subnets[*].SubnetId" --output text --region ${AWS_REGION} | tr "\\t" ",")
                    aws cloudformation deploy \
                        --template-file templates/alb-template.yaml \
                        --stack-name ${STACK_PREFIX}-ALB \
                        --region ${AWS_REGION} \
                        --parameter-overrides VpcId=$VPC_ID Subnets=$SUBNETS \
                        --capabilities CAPABILITY_NAMED_IAM \
                        --no-fail-on-empty-changeset
                    '''
                }
            }
        }

        stage('Deploy NLB Stack') {
            steps {
                echo "⚡ Deploying NLB Stack..."
                withAWS(region: "${AWS_REGION}", credentials: 'aws-cred') {
                    sh '''
                    VPC_ID=$(aws ec2 describe-vpcs --query "Vpcs[0].VpcId" --output text --region ${AWS_REGION})
                    SUBNETS=$(aws ec2 describe-subnets --query "Subnets[*].SubnetId" --output text --region ${AWS_REGION} | tr "\\t" ",")
                    aws cloudformation deploy \
                        --template-file templates/nlb-template.yaml \
                        --stack-name ${STACK_PREFIX}-NLB \
                        --region ${AWS_REGION} \
                        --parameter-overrides VpcId=$VPC_ID Subnets=$SUBNETS \
                        --capabilities CAPABILITY_NAMED_IAM \
                        --no-fail-on-empty-changeset
                    '''
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
