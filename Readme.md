1x ALB and TargetGroup should be there.
1x docker file that you can compile and push from local system to AWS ECR.


Step 1.
docker logout public.ecr.aws
aws ecr get-login-password --region eu-west-1 --profile AWS_PROFILE | docker login --username AWS --password-stdin AWS_ACCT_ID.dkr.ecr.eu-west-1.amazonaws.com/base-repo

Step 2.
aws ecr create-repository --repository-name base-repo --image-tag-mutability IMMUTABLE
docker build -t base-repo .

docker tag base-repo:latest AWS_ACCT_ID.dkr.ecr.eu-west-1.amazonaws.com/base-repo:latest
docker push AWS_ACCT_ID.dkr.ecr.eu-west-1.amazonaws.com/base-repo:latest

Step 3.
aws ecs register-task-definition --cli-input-json file://TaskDefinition.json --region eu-west-1 --query 'taskDefinition.taskDefinitionArn' --profile citrus_lab

aws ecs create-cluster --cluster-name base-repo --region eu-west-1 --profile AWS_PROFILE

aws ecs create-service --cli-input-json file://service.json  --region eu-west-1 --profile AWS_PROFILE

Step 4.
Add autoscaling:
aws application-autoscaling register-scalable-target --resource-id service/base-repo/base-repo --service-namespace ecs --scalable-dimension ecs:service:DesiredCount --min-capacity 1 --max-capacity 20 --role-arn arn:aws:iam::AWS_ACCT_ID:role/ecsServiceAutoScalingrole --region eu-west-1 --profile AWS_PROFILE

aws application-autoscaling put-scaling-policy --cli-input-json file://scale-out.json --region eu-west-1 --profile AWS_PROFILE