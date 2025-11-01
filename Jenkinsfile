pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        KEY_NAME = 'sanjana-key'
    }

    stages {

        stage('Deploy EC2 Stack') {
            steps {
                echo '🚀 Deploying EC2 Stack...'
                withAWS(region: "${AWS_REGION}") {
                    sh """
                    aws cloudformation deploy \
                        --template-file templates/ec2-template.yaml \
                        --stack-name NetworkInfraStack-EC2 \
                        --region ${AWS_REGION} \
                        --parameter-overrides KeyName=${KEY_NAME} \
                        --capabilities CAPABILITY_NAMED_IAM \
                        --no-fail-on-empty-changeset
                    """
                }
            }
        }

        stage('Deploy ALB Stack') {
            steps {
                echo '🌐 Deploying ALB Stack...'
                withAWS(region: "${AWS_REGION}") {
                    script {
                        def vpcId = sh(
                            script: "aws ec2 describe-vpcs --query 'Vpcs[0].VpcId' --output text --region ${AWS_REGION}",
                            returnStdout: true
                        ).trim()

                        def subnets = sh(
                            script: "aws ec2 describe-subnets --filters Name=vpc-id,Values=${vpcId} --query 'Subnets[0:2].SubnetId' --output text --region ${AWS_REGION} | tr '\\t' ','",
                            returnStdout: true
                        ).trim()

                        echo "Using VPC ID: ${vpcId}"
                        echo "Using Subnets: ${subnets}"

                        sh """
                        aws cloudformation deploy \
                            --template-file templates/alb-template.yaml \
                            --stack-name NetworkInfraStack-ALB \
                            --region ${AWS_REGION} \
                            --parameter-overrides VpcId=${vpcId} Subnets=${subnets} \
                            --capabilities CAPABILITY_NAMED_IAM \
                            --no-fail-on-empty-changeset
                        """
                    }
                }
            }
            post {
                failure {
                    echo "❌ ALB Stack deployment failed. Fetching error details..."
                    sh "aws cloudformation describe-stack-events --stack-name NetworkInfraStack-ALB --region ${AWS_REGION} --max-items 5"
                }
            }
        }

        stage('Deploy NLB Stack') {
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                echo '⚡ Deploying NLB Stack...'
                withAWS(region: "${AWS_REGION}") {
                    sh """
                    aws cloudformation deploy \
                        --template-file templates/nlb-template.yaml \
                        --stack-name NetworkInfraStack-NLB \
                        --region ${AWS_REGION} \
                        --capabilities CAPABILITY_NAMED_IAM \
                        --no-fail-on-empty-changeset
                    """
                }
            }
        }
    }

    post {
        failure {
            echo "❌ Deployment failed. Please check the CloudFormation console."
        }
        success {
            echo "✅ All stacks deployed successfully!"
        }
    }
}
