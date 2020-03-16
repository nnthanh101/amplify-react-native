---
title: Cài Đặt
weight: 15
pre: "<b>1.2. </b>"
---

In the Cloud9 workspace, run the following commands:

- First install the AWS Amplify React Native CLI (plus some other text utilities):

```
sudo curl -so /usr/local/bin/ecs-cli https://s3.amazonaws.com/amazon-ecs-cli/ecs-cli-linux-amd64-latest
sudo chmod +x /usr/local/bin/ecs-cli

sudo yum -y install jq gettext
```

- Next configure the AWS cli with our current region as default:

```
export AWS_REGION=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r .region)
echo "export AWS_REGION=${AWS_REGION}" >> ~/.bash_profile
aws configure set default.region ${AWS_REGION}
aws configure get default.region
```