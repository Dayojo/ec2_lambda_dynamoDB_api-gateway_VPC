# ec2_lambda_dynamoDB_api-gateway_VPC
AWS : ec2, lambda, dynamoDB, api-gateway, VPC

# AWS Lambda Infrastructure Project

This project provisions a complete AWS infrastructure using Terraform, including Lambda, EC2, DynamoDB, API Gateway, and VPC.

## Table of Contents
1. [Initial AWS Setup](#initial-aws-setup)
2. [Architecture Overview](#architecture-overview)
3. [Resource Configuration Steps](#resource-configuration-steps)
4. [Deployment Guide](#deployment-guide)
5. [Testing and Verification](#testing-and-verification)
6. [Monitoring and Maintenance](#monitoring-and-maintenance)
7. [Troubleshooting](#troubleshooting)
8. [Cost Considerations](#cost-considerations)

## Initial AWS Setup

### AWS Console Access
1. Navigate to [AWS Console](https://console.aws.amazon.com)
2. Sign in with your AWS account credentials
3. Select region "us-east-2" from the top-right dropdown

### AWS CLI Configuration
1. Install AWS CLI:
   ```bash
   # macOS (using Homebrew)
   brew install awscli

   # Windows (using installer)
   # Download and run AWS CLI MSI installer
   ```

2. Configure AWS CLI:
   ```bash
   aws configure
   ```
   Enter:
   - AWS Access Key ID
   - AWS Secret Access Key
   - Default region: us-east-2
   - Default output format: json

## Architecture Overview

### 1. VPC Configuration
- [VPC Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
- Components:
  * VPC with CIDR 10.0.0.0/16
  * Public and private subnets
  * Internet Gateway
  * NAT Gateway
  * Route tables
  * Security groups

### 2. Lambda Function
- [Lambda Documentation](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
- Features:
  * Node.js 18.x runtime
  * VPC integration
  * CloudWatch logging
  * IAM roles and policies

### 3. DynamoDB
- [DynamoDB Documentation](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html)
- Configuration:
  * On-demand capacity
  * Point-in-time recovery
  * Server-side encryption
  * Auto-scaling capabilities

### 4. API Gateway
- [API Gateway Documentation](https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html)
- Setup:
  * REST API
  * Lambda integration
  * CloudWatch logging
  * Multiple stages (dev/prod)

### 5. EC2 Instance
- [EC2 Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)
- Features:
  * Amazon Linux 2023
  * Private subnet placement
  * Security group configuration
  * CloudWatch monitoring

## Resource Configuration Steps

### VPC Setup

#### Console Steps:
1. Navigate to VPC Dashboard:
   - Open AWS Console → Search "VPC" → Click "Your VPCs"
   - Click "Create VPC"

2. Configure VPC:
   - Name: aws-lambda-project-vpc
   - IPv4 CIDR: 10.0.0.0/16
   - Click "Create VPC"

3. Create Subnets:
   - Click "Subnets" → "Create subnet"
   - Public subnets:
     * Name: public-subnet-1
     * CIDR: 10.0.1.0/24
     * AZ: us-east-2a
   - Private subnets:
     * Name: private-subnet-1
     * CIDR: 10.0.10.0/24
     * AZ: us-east-2a

#### Terraform Configuration:
```hcl
# vpc/main.tf
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true
}
```

### Lambda Function Setup

#### Console Steps:
1. Navigate to Lambda:
   - AWS Console → Search "Lambda"
   - Click "Create function"

2. Basic Configuration:
   - Choose "Author from scratch"
   - Name: aws-lambda-project-function
   - Runtime: Node.js 18.x
   - Architecture: x86_64

3. VPC Configuration:
   - Scroll to "VPC" section
   - Select created VPC
   - Choose private subnets
   - Select Lambda security group

#### Terraform Configuration:
```hcl
# lambda/main.tf
resource "aws_lambda_function" "main" {
  filename      = "function.zip"
  function_name = "aws-lambda-project-function"
  role          = aws_iam_role.lambda_role.arn
  handler       = "index.handler"
  runtime       = "nodejs18.x"
}
```

### DynamoDB Setup

#### Console Steps:
1. Navigate to DynamoDB:
   - AWS Console → Search "DynamoDB"
   - Click "Create table"

2. Table Configuration:
   - Table name: aws-lambda-project-table
   - Partition key: id (String)
   - Sort key: status (String)

3. Additional Settings:
   - Capacity mode: On-demand
   - Encryption: AWS owned key
   - Enable point-in-time recovery

#### Terraform Configuration:
```hcl
# dynamodb/main.tf
resource "aws_dynamodb_table" "main" {
  name           = "aws-lambda-project-table"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "id"
  range_key      = "status"
}
```

### API Gateway Setup

#### Console Steps:
1. Navigate to API Gateway:
   - AWS Console → Search "API Gateway"
   - Click "Create API"

2. API Configuration:
   - Choose "REST API"
   - Name: aws-lambda-project-api
   - Click "Create API"

3. Resource/Method Setup:
   - Create Resource: "items"
   - Add Methods: GET, POST
   - Configure Lambda integration

#### Terraform Configuration:
```hcl
# api-gateway/main.tf
resource "aws_api_gateway_rest_api" "main" {
  name = "aws-lambda-project-api"
  description = "API Gateway for Lambda project"
}
```

### EC2 Instance Setup

#### Console Steps:
1. Navigate to EC2:
   - AWS Console → Search "EC2"
   - Click "Launch Instance"

2. Instance Configuration:
   - Name: aws-lambda-project-ec2
   - AMI: Amazon Linux 2023
   - Instance type: t3.micro
   - VPC: Select created VPC
   - Subnet: Select private subnet

#### Terraform Configuration:
```hcl
# ec2/main.tf
resource "aws_instance" "main" {
  ami           = "ami-0430580de6244e02e"
  instance_type = "t3.micro"
  subnet_id     = var.subnet_id
}
```

## Deployment Guide

### Prerequisites
1. AWS CLI installed and configured
2. Terraform (version >= 1.0) installed
3. AWS account with appropriate permissions
4. SSH key pair for EC2 instance access

### Deployment Steps

1. Clone and Configure:
```bash
git clone <repository-url>
cd AWS_lambda
cp terraform.tfvars.example terraform.tfvars
```

2. Edit terraform.tfvars:
```hcl
project_name = "your-project-name"
environment  = "dev"
key_name     = "your-key-pair-name"
```

3. Deploy Infrastructure:
```bash
chmod +x deploy.sh
./deploy.sh
```

## Testing and Verification

### 1. VPC Verification
- Check VPC in AWS Console
- Verify subnet configurations
- Test network connectivity

### 2. Lambda Testing
- Use AWS Console test feature
- Check CloudWatch logs
- Test VPC connectivity

### 3. DynamoDB Verification
- Verify table creation
- Test CRUD operations
- Check auto-scaling settings

### 4. API Gateway Testing
```bash
# Test GET endpoint
curl -X GET https://your-api-id.execute-api.us-east-2.amazonaws.com/dev/items

# Test POST endpoint
curl -X POST https://your-api-id.execute-api.us-east-2.amazonaws.com/dev/items \
  -H "Content-Type: application/json" \
  -d '{"id": "test", "status": "active"}'
```

### 5. EC2 Verification
- Check instance status
- Verify security group rules
- Test SSH access (through bastion if applicable)

## Monitoring and Maintenance

### CloudWatch Monitoring
1. Access CloudWatch:
   - AWS Console → CloudWatch
   - Check Lambda function logs
   - Monitor API Gateway metrics
   - View EC2 metrics

### Cost Management
1. AWS Cost Explorer:
   - Monitor resource costs
   - Set up budget alerts
   - Review usage patterns

### Security Updates
1. Regular Tasks:
   - Review security groups
   - Update Lambda runtimes
   - Check AWS advisories
   - Monitor CloudTrail

## Troubleshooting

### Common Issues and Solutions

1. VPC Issues:
   - Check route tables
   - Verify security group rules
   - Confirm NAT Gateway status

2. Lambda Issues:
   - Review CloudWatch logs
   - Check IAM permissions
   - Verify VPC configuration

3. API Gateway Issues:
   - Validate Lambda integration
   - Check endpoint configuration
   - Review CORS settings

## Cost Considerations

Detailed pricing links:
- [VPC Pricing](https://aws.amazon.com/vpc/pricing/)
- [Lambda Pricing](https://aws.amazon.com/lambda/pricing/)
- [DynamoDB Pricing](https://aws.amazon.com/dynamodb/pricing/)
- [API Gateway Pricing](https://aws.amazon.com/api-gateway/pricing/)
- [EC2 Pricing](https://aws.amazon.com/ec2/pricing/)

