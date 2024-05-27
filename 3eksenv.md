# Switching between different EKS Cluster in your Cloud9 Bash Terminals

Open up 3 bash terminals in your cloud9 Environment; Set the variables for each so that you can log in

Dev Environment - us-west-2

```
export AWS_REGION=us-west-2
export KUBE_CONFIG=$(aws cloudformation describe-stacks --stack-name dev-dev-blueprint | jq -r '.Stacks[0].Outputs[] | select(.OutputKey|match("ConfigCommand"))| .OutputValue')
$KUBE_CONFIG
```

Test enviroment- us-east-2
```
export AWS_REGION=us-east-2
export KUBE_CONFIG=$(aws cloudformation describe-stacks --stack-name test-test-blueprint | jq -r '.Stacks[0].Outputs[] | select(.OutputKey|match("ConfigCommand"))| .OutputValue')
$KUBE_CONFIG
```

Production Environment - us-east-1
```
export AWS_REGION=us-east-1
export KUBE_CONFIG=$(aws cloudformation describe-stacks --stack-name prod-prod-blueprint | jq -r '.Stacks[0].Outputs[] | select(.OutputKey|match("ConfigCommand"))| .OutputValue')
$KUBE_CONFIG
```

Useful Commands
Check current cluster context
```
kubectl config current-context
```
```
aws sts get-caller-identity
```
