+++
title = "Deploy the NodeJS Service"
chapter = false
weight = 20
pre = "<b>2.2. </b>"
+++

Let’s bring up the NodeJS Backend API!

#### Deploy our NodeJS Backend Application:
```
cd ~/environment/amplify-react-native-nodejs
envsubst < ecs-params.yml.template >ecs-params.yml

ecs-cli compose --project-name amplify-react-native-nodejs service up \
    --create-log-groups \
    --private-dns-namespace service \
    --enable-service-discovery \
    --cluster-config amplify-react-native \
    --vpc $vpc
    
```
Here, we change directories into our nodejs application code directory.
The `envsubst` command templates our `ecs-params.yml` file with our current values.
We then launch our nodejs service on our AWS Amplify React Native cluster (with a default launchtype 
of Fargate)

Note: ecs-cli will take care of building our private dns namespace for service discovery,
and log group in cloudwatch logs.

#### View running container, and store the output of the task id as an env variable for later use:
```
ecs-cli compose --project-name amplify-react-native-nodejs service ps \
    --cluster-config amplify-react-native

task_id=$(ecs-cli compose --project-name amplify-react-native-nodejs service ps --cluster-config amplify-react-native | awk -F \/ 'FNR == 2 {print $1}')
```
We should have one task registered.

#### Check reachability (open url in your browser):
```
alb_url=$(aws cloudformation describe-stacks --stack-name amplify-react-native-alb --query 'Stacks[0].Outputs[?OutputKey==`ExternalUrl`].OutputValue' --output text)
echo "Open $alb_url in your browser"
```
This command looks up the URL for our ingress ALB, and outputs it. You should 
be able to click to open, or copy-paste into your browser.

#### View logs:
```
# Referencing task id from above ps command
ecs-cli logs --task-id $task_id \
    --follow --cluster-config amplify-react-native
```
To view logs, find the task id from the earlier `ps` command, and use it in this
command. You can follow a task's logs also.

#### Scale the tasks:
```
ecs-cli compose --project-name amplify-react-native-nodejs service scale 3 \
    --cluster-config amplify-react-native
ecs-cli compose --project-name amplify-react-native-nodejs service ps \
    --cluster-config amplify-react-native
```
We can see that our containers have now been evenly distributed across all 3 of our
availability zones.