---
title: Build the Platform
weight: 10
pre: "<b>2.1. </b>"
---

Breaking down a Monolith into Micro-Services is a good idea ⛅
We will use 2 CloudFormation Stacks that will build our AWS Amplify React Native Cluster and ALB.

### Ensure service linked roles exist for **Load Balancers** and **AWS Amplify React Native**:

```
aws iam get-role --role-name "AWSServiceRoleForElasticLoadBalancing" || aws iam create-service-linked-role --aws-service-name "elasticloadbalancing.amazonaws.com"

aws iam get-role --role-name "AWSServiceRoleForECS" || aws iam create-service-linked-role --aws-service-name "ecs.amazonaws.com"
```


### Build a **VPC**, **AWS Amplify React Native Cluster**, and **ALB**: ecs-cli fargate

```
cd ~/environment/amplify-react-native

aws cloudformation deploy --stack-name amplify-react-native --template-file cluster-fargate-private-vpc.yml --capabilities CAPABILITY_IAM

aws cloudformation deploy --stack-name amplify-react-native-alb --template-file alb-external.yml
```

> {{%expand "ecs-cli ec2 mode" %}}
```
cd ~/environment/amplify-react-native

aws cloudformation deploy --stack-name amplify-react-native --template-file cluster-ec2-private-vpc.yml --capabilities CAPABILITY_IAM

aws cloudformation deploy --stack-name amplify-react-native-alb --template-file alb-external.yml
```
{{% /expand%}}


### Set environment variables from our build

```
export clustername=$(aws cloudformation describe-stacks --stack-name amplify-react-native --query 'Stacks[0].Outputs[?OutputKey==`ClusterName`].OutputValue' --output text)
export target_group_arn=$(aws cloudformation describe-stack-resources --stack-name amplify-react-native-alb | jq -r '.[][] | select(.ResourceType=="AWS::ElasticLoadBalancingV2::TargetGroup").PhysicalResourceId')
export vpc=$(aws cloudformation describe-stacks --stack-name amplify-react-native --query 'Stacks[0].Outputs[?OutputKey==`VpcId`].OutputValue' --output text)
export ecsTaskExecutionRole=$(aws cloudformation describe-stacks --stack-name amplify-react-native --query 'Stacks[0].Outputs[?OutputKey==`ECSTaskExecutionRole`].OutputValue' --output text)
export subnet_1=$(aws cloudformation describe-stacks --stack-name amplify-react-native --query 'Stacks[0].Outputs[?OutputKey==`PrivateSubnetOne`].OutputValue' --output text)
export subnet_2=$(aws cloudformation describe-stacks --stack-name amplify-react-native --query 'Stacks[0].Outputs[?OutputKey==`PrivateSubnetTwo`].OutputValue' --output text)
export subnet_3=$(aws cloudformation describe-stacks --stack-name amplify-react-native --query 'Stacks[0].Outputs[?OutputKey==`PrivateSubnetThree`].OutputValue' --output text)
export security_group=$(aws cloudformation describe-stacks --stack-name amplify-react-native --query 'Stacks[0].Outputs[?OutputKey==`ContainerSecurityGroup`].OutputValue' --output text)
```
This creates our infrastructure, and sets several environment variables we will use to
automate deploys.

### Configure `ecs-cli` to talk to your cluster:

```
ecs-cli configure --region $AWS_REGION --cluster $clustername --default-launch-type FARGATE --config-name amplify-react-native
```
We set a default region so we can reference the region when we run our commands.

### Authorize traffic:

```
aws ec2 authorize-security-group-ingress --group-id "$security_group" --protocol tcp --port 3000 --cidr 0.0.0.0/0
```
We know that our containers talk on port 3000, so authorize that traffic on our security group.