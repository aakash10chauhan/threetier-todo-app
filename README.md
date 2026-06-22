## This document is a guide to implement an architecture on AWS cloud, EKS cluster and resources are managed by Fargate using fargate profile

sudo apt install awscli -y
# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl && sudo mv kubectl /usr/local/bin/

# Install eksctl
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz"
tar -xzf eksctl_Linux_amd64.tar.gz && sudo mv eksctl /usr/local/bin/

aws configure

Step 1 — Create the EKS cluster with Fargate

eksctl create cluster --name my-cluster --region us-east-1 --fargate --alb-ingress-access

aws eks update-kubeconfig --region us-east-1 --name my-cluster

Step 2 — Create a Fargate profile for your app namespace

Fargate profile "three-tier-profile" on EKS cluster "my-cluster"

eksctl create fargateprofile --cluster my-cluster --name three-tier-profile --namespace three-tier --region us-east-1

# Associate OIDC provider (needed for IAM roles for service accounts)

eksctl utils associate-iam-oidc-provider --cluster my-cluster --region us-east-1 --approve

# Download the IAM policy

curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json

# Create the IAM policy
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json

# Create the service account (replace 252610334383 with your AWS account ID)
eksctl create iamserviceaccount --cluster my-cluster --namespace kube-system --name aws-load-balancer-controller --attach-policy-arn arn:aws:iam::252610334383:policy/AWSLoadBalancerControllerIAMPolicy --override-existing-serviceaccounts --approve --region us-east-1

download and update helm

curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4 | bash

helm repo add eks https://aws.github.io/eks-charts

helm repo update

helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=my-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller --set region=us-east-1 --set vpcId=$(aws eks describe-cluster --name my-cluster --query "cluster.resourcesVpcConfig.vpcId" --output text)

now kubectl apply -f <your-k8s-deployments/services.yaml>

delete after your deployment is successful

eksctl delete cluster --name my-cluster --region us-east-1

eksctl delete iamserviceaccount --cluster my-cluster --namespace kube-system --name aws-load-balancer-controller --region us-east-1

eksctl delete fargateprofile --cluster my-cluster --name three-tier-profile --region us-east-1

aws eks delete-fargate-profile --cluster-name my-cluster --fargate-profile-name three-tier-profile --region us-east-1




----------------------------------Post deployment steps if required--------------------------------------
aws iam create-policy-version --policy-arn arn:aws:iam::252610334383:policy/AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json --set-as-default

new policy
curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json

find existing policy

aws iam list-policies --query "Policies[?PolicyName=='AWSLoadBalancerControllerIAMPolicy'].Arn" --output text

then replace with downloaded policy

aws iam create-policy-version --policy-arn <YOUR_POLICY_ARN> --policy-document file://iam_policy.json --set-as-default

