# Preparing the EKS Cluster 
These steps will help in preparing the EKS CLuster for installation of Kasten in a AWS Cloud9 Environment

## Prerequisite
In the labs, there are 3 cluster in 3 regions. dev-blueprint in us-west-2; test-blueprint in us-east-2; prod-blueprint in us-east-1. It will be easier to create global variables each time
```
cluster_name=dev-blueprint
```

## Install Tools
Install eksctl
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo cp /tmp/eksctl /usr/bin
eksctl version
```
Install helm
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
helm version
```
Install kubectl if needed
```
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.18.9/2020-11-02/bin/linux/amd64/kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
kubectl version --short --client
```
Install jq if needed
```
sudo yum install jq
```
## Switching kubectl context
Go to ..... and then switch context to your required one; The first one will be dev environment which will be similar to the command below; 
You will need to switch to Prod environment when redoing this for the 2nd EKS Cluster
```
export AWS_REGION=us-west-2
export KUBE_CONFIG=$(aws cloudformation describe-stacks --stack-name dev-dev-blueprint | jq -r '.Stacks[0].Outputs[] | select(.OutputKey|match("ConfigCommand"))| .OutputValue')
$KUBE_CONFIG
```
Check you are in the right enviroment
```
kubectl config current-context
```

## Create an IAM OIDC provider for your cluster

It will allow kubernetes service account to authentify to IAM and assume roles to do the CSI operations. Please change the cluster_name to your exisitng cluster

```
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```
## Create the Amazon EBS CSI driver IAM role
```
eksctl create iamserviceaccount \
    --name ebs-csi-controller-sa \
    --namespace kube-system \
    --cluster $cluster_name \
    --role-name "k10_${cluster_name}_AWS_EBS_CSI_DriverRole" \
    --role-only \
    --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
    --approve
```
## Install the EBS Add On, Create Storage Class and Test it
Go to the AWS Console. Go to your EKS cluster. Click on dev-blueprint. Click on Add-ons. Click Get more add-ons. Add your Amazon EBS CSI Driver and then add the above role to it.

Check the Status
```
eksctl get addon --name aws-ebs-csi-driver --cluster $cluster_name
```
Look at the pods in kube-system; EBS Pods should be there
```
kubectl get pods -n kube-system
```
Deploy the Storage Class and provisioned a sample app with a pvc ebs-claim
```
git clone https://github.com/kubernetes-sigs/aws-ebs-csi-driver.git
cd aws-ebs-csi-driver/examples/kubernetes/dynamic-provisioning/
kubectl apply -f manifests/
kubectl describe storageclass ebs-sc
```
Check pvc is created
```
kubectl get pvc
```
Delete pvc; If you can't delete the pvc, run this command in another terminal; 
kubectl patch pvc ebs-claim -p'{"metadata":{"finalizers":null}}'

```
kubectl delete app -n default
kubectl delete pvc ebs-claim
```
Check Available StorageClass
```
kubectl get sc
```
Make EBS the default storage class, check the storage class again after running the commands below
```
kubectl annotate storageclass gp2 storageclass.kubernetes.io/is-default-class=false --overwrite
kubectl annotate storageclass ebs-sc storageclass.kubernetes.io/is-default-class=true
```
## Deploy snapshot controller
```
cd ../../../..
```
check the last version of the https://github.com/kubernetes-csi/external-snapshotter project.
In our case it's v7.0.2.
```
version=v7.0.2
```
Install the CRD
```
kubectl apply -k "github.com/kubernetes-csi/external-snapshotter/client/config/crd?ref=$version"
```
Install the snapshot controller
```
kubectl apply -k "github.com/kubernetes-csi/external-snapshotter/deploy/kubernetes/snapshot-controller?ref=$version"
```
Create a volumesnapshotclass
```
cat <<EOF | kubectl create -f -
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshotClass
metadata:
  name: ebs-snapshotclass
driver: ebs.csi.aws.com
deletionPolicy: Delete
EOF
```
Annotate the volumesnapshotclass
```
kubectl annotate volumesnapshotclass ebs-snapshotclass k10.kasten.io/is-snapshot-class=true
```

Please proceed to the next step to install Kasten K10<br>
 (https://github.com/munsw/K10-and-EKS-Blueprints-Lab/new/main#installing-kasten-k10)




