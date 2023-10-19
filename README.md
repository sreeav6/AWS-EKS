# Here we will deploy application to EKS
Prerequisities that you should have kubectl and eksctl and aws cli installed in your system

# First we will create eksctl cluster 
eksctl create cluster --name <name of your cluster> --region us-east-1 --fargate 

# Then we will update the kube config for aws to use eks
aws eks update-kubeconfig --name <name of your cluster> --region us-east-1
 
# Create a seperate fargate profile where we use this namespace to deploy the application
eksctl create fargateprofile --cluster <name of your cluster> --region us-east-1 --name alb-sample-app --namespace game-2048

# Deploy the app which has deployment,service,ingress manifests
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml

#you can verify this with kubectl get pods -n game-2048

#you can verify the service kubectl get svc -n game-2048

#you can verify the deployments kubectl get deployments -n game-2048

#you can verify the ingress resource kubectk get ingress -n game-2048

#you can verify all kubectl get all -n game-2048

# Use OIDC connector as AWS IAM

eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve

# Setting alb and add on

#We need to download the iam policy first

curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json

# We need to create iam policy

aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json

# We will be creating iam role

eksctl create iamserviceaccount \
  --cluster= <your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve

# Deploy the alb

#ADD helm repo

helm repo add eks https://aws.github.io/eks-charts

#Update the repo

helm repo update eks

#Install alb using helm

helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
  --set clusterName= <your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=us-east-1 \
  --set vpcId=<your-vpc-id>

# Verify the deployment is running

  kubectl get deployment -n kube-system aws-load-balancer-controller
  
  #you can also see the ingress class once alb is deployed
  
  kubectl get ingress -n game-2048
  
# Finally perform deletion (clean up)

eksctl delete cluster --name <your-cluster-name> --region us-east-1
