pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_CREDENTIALS = 'aws-credentials'   /
        STACK_PREFIX = 'NetworkInfraStack'
    }

    stages {
        stage('📦 Checkout Code') {
            steps {
                echo 'Checking out infrastructure templates from GitHub...'
                git branch: 'main', url: ''
            }
        }

        stage('🚀 Deploy EC2 Stack') {
            steps {
                withAWS(region: "${AWS_REGION}", credentials: "${AWS_CREDENTIALS}") {
                    echo 'Deploying EC2 stack...'
                    sh '''
                        aws cloudformation deploy \
                          --template-file ec2-template.yaml \
                          --stack-name ${STACK_PREFIX}-EC2 \
                          --parameter-overrides KeyName=my-key InstanceType=t2.micro \
                          --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM \
                          --region ${AWS_REGION}
                    '''
                }
            }
        }

        stage('🚀 Deploy ALB Stack') {
            steps {
                withAWS(region: "${AWS_REGION}", credentials: "${AWS_CREDENTIALS}") {
                    echo 'Deploying ALB stack...'
                    sh '''
                        aws cloudformation deploy \
                          --template-file alb-template.yaml \
                          --stack-name ${STACK_PREFIX}-ALB \
                          --parameter-overrides VpcId=vpc-xxxxxxx Subnets=subnet-aaaaaaa,subnet-bbbbbbb \
                          --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM \
                          --region ${AWS_REGION}
                    '''
                }
            }
        }

        stage('🚀 Deploy NLB Stack') {
            steps {
                withAWS(region: "${AWS_REGION}", credentials: "${AWS_CREDENTIALS}") {
                    echo 'Deploying NLB stack...'
                    sh '''
                        aws cloudformation deploy \
                          --template-file nlb-template.yaml \
                          --stack-name ${STACK_PREFIX}-NLB \
                          --parameter-overrides VpcId=vpc-xxxxxxx Subnets=subnet-aaaaaaa,subnet-bbbbbbb \
                          --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM \
                          --region ${AWS_REGION}
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ All AWS stacks deployed successfully!'
        }
        failure {
            echo '❌ Deployment failed. Check the CloudFormation or Jenkins logs.'
        }
    }
}
