# Easy-ECS

A streamlined toolkit for deploying containerized applications to AWS Elastic Container Service (ECS) with Fargate.

## Overview

Easy-ECS provides a set of configuration templates and step-by-step instructions to quickly deploy Docker containers to AWS ECS Fargate. The project includes:

- Task definition templates
- Service configuration
- Auto-scaling policies
- Load testing utilities

## Prerequisites

- AWS CLI configured with appropriate permissions
- Docker installed locally
- An AWS account with access to:
  - Amazon ECR (Elastic Container Registry)
  - Amazon ECS (Elastic Container Service)
  - AWS IAM (Identity and Access Management)
  - Amazon VPC (Virtual Private Cloud)
  - AWS Application Auto Scaling

## Deployment Steps

### 1. Authenticate with Amazon ECR

```bash
docker logout public.ecr.aws
aws ecr get-login-password --region eu-west-1 --profile YOUR_AWS_PROFILE | docker login --username AWS --password-stdin YOUR_AWS_ACCOUNT_ID.dkr.ecr.eu-west-1.amazonaws.com/base-repo
```

### 2. Create Repository and Push Docker Image

```bash
# Create ECR repository
aws ecr create-repository --repository-name base-repo --image-tag-mutability IMMUTABLE

# Build Docker image
docker build -t base-repo .

# Tag and push to ECR
docker tag base-repo:latest YOUR_AWS_ACCOUNT_ID.dkr.ecr.eu-west-1.amazonaws.com/base-repo:latest
docker push YOUR_AWS_ACCOUNT_ID.dkr.ecr.eu-west-1.amazonaws.com/base-repo:latest
```

### 3. Deploy ECS Resources

```bash
# Register task definition
aws ecs register-task-definition --cli-input-json file://TaskDefinition.json --region eu-west-1 --query 'taskDefinition.taskDefinitionArn' --profile YOUR_AWS_PROFILE

# Create ECS cluster
aws ecs create-cluster --cluster-name base-repo --region eu-west-1 --profile YOUR_AWS_PROFILE

# Create ECS service
aws ecs create-service --cli-input-json file://service.json --region eu-west-1 --profile YOUR_AWS_PROFILE
```

### 4. Configure Auto-scaling

```bash
# Register scalable target
aws application-autoscaling register-scalable-target \
  --resource-id service/base-repo/base-repo \
  --service-namespace ecs \
  --scalable-dimension ecs:service:DesiredCount \
  --min-capacity 1 \
  --max-capacity 20 \
  --role-arn arn:aws:iam::YOUR_AWS_ACCOUNT_ID:role/ecsServiceAutoScalingrole \
  --region eu-west-1 \
  --profile YOUR_AWS_PROFILE

# Apply scaling policy
aws application-autoscaling put-scaling-policy --cli-input-json file://scale-out.json --region eu-west-1 --profile YOUR_AWS_PROFILE
```

### 5. Load Testing

Store your ALB URL in a variable and run load tests:

```bash
# Simple continuous requests
while true; do curl -so /dev/null $alb_url; sleep 1; done &

# Apache Bench load testing
while true; do ab -l -c 9 -t 1200 curl -so /dev/null $alb_url; sleep 1; done &
```

## Configuration Files

### TaskDefinition.json

Defines the Fargate task with CPU, memory, networking, and container specifications.

### service.json

Configures the ECS service with load balancer integration and networking settings.

### scale-out.json

Defines auto-scaling policies based on CPU utilization metrics.

## Notes

- Before deployment, update all placeholder values (YOUR_AWS_PROFILE, YOUR_AWS_ACCOUNT_ID) in the commands and configuration files
- Ensure your AWS account has the necessary IAM roles created (ecsTaskExecutionRole, ecsServiceAutoScalingrole)
- The default configuration uses private subnets with no public IP assignment
- Auto-scaling is configured to trigger when CPU utilization exceeds 50%

## Requirements

- AWS CLI version 2.0+
- Docker 19.03+
- Apache Bench (ab) for load testing
