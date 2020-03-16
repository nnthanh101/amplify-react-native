---
title: Build the Platform
weight: 20
pre: "<b>2.1. </b>"
---

Breaking down a Monolith into Micro-Services is a good idea ⛅
We will use 2 CloudFormation Stacks that will build our AWS Amplify React Native Cluster and ALB.

> Ensure service linked roles exist for **Load Balancers** and **AWS Amplify React Native**:

```
aws iam get-role --role-name "AWSServiceRoleForElasticLoadBalancing" || aws iam create-service-linked-role --aws-service-name "elasticloadbalancing.amazonaws.com"

aws iam get-role --role-name "AWSServiceRoleForECS" || aws iam create-service-linked-role --aws-service-name "ecs.amazonaws.com"
```


> Build a **VPC**, **AWS Amplify React Native Cluster**, and **ALB**: ecs-cli fargate

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