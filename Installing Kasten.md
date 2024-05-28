# Installing Kasten K10

## Install Requirements
Add the Kasten Helm charts repository using:
```
helm repo add kasten https://charts.kasten.io/
```
## Create Kasten namespace
```
kubectl create namespace kasten-io
```
## Preflight Check for Kasten
```
curl -s https://docs.kasten.io/tools/k10_primer.sh | bash /dev/stdin csi -s ebs-sc -n default -u 1001
```
## Install Kasten
This will install Kasten with the Role as called from below. Set the authethication token, use AWS ELB and do not provision promethus and grafana due to low resources in the lab
```
aws sts get-caller-identity
```
Copy the role from the command above and put it in the --set secrets.awsIamRole="arn:aws:iam::(copy the account id and paste)"
```
helm install k10 kasten/k10 --namespace=kasten-io \
--set secrets.awsIamRole="arn:aws:iam::790845121320:role/eks-blueprints-cdk-workshop-admin-20fa4ab0" \
--set auth.tokenAuth.enabled=true --set externalGateway.create=true \
--set "prometheus.server.enabled=false" --set "grafana.enabled=false"
```
Watch Kasten Pods come up
```
kubectl get pods -n kasten-io -w
```
## Accessing Kasten
Get the URL
```
kubectl get svc gateway-ext --namespace kasten-io -o wide
```
Copy the URL and then add the prefix /k10/# at the back. Example: `http://SERVICE_EXTERNAL_IP/k10/#/`

Get the Token to access
```
kubectl --namespace kasten-io create token k10-k10 --duration=24h
```
## Pacman
Install Pacman and deploy with AWS ELB

Add Pacman repo
```
helm repo add pacman https://shuguet.github.io/pacman/
```
Install Pacman
```
helm install pacman pacman/pacman -n pacman --create-namespace \
    --set service.type=LoadBalancer
```


