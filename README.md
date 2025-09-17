## Create EKS Cluster on AWS
```
eksctl create cluster --name demo-cluster --region us-east-1 --fargate
```
![alt text](image.png)

# update kubeconfig file
```
aws eks update-kubeconfig --name demo-cluster --region us-east-1
```
# create a name to deploy 2048 game
```
eksctl create fargateprofile --cluster demo-cluster --region us-east-1 --name alb-sample-app --namespace game-2048
```

# create namespace, deployment, service, ingress
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```
# Configure IAM OIDC Provider
```
eksctl utils associate-iam-oidc-provider --cluster demo-cluster --approve
```
# download iam_policy json file
```
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json
```

# Create IAM Policy
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json

# Create IAM ROlE
```
eksctl create iamserviceaccount \
  --cluster=<your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```
# Deploy ALB controller
```
helm repo add eks https://aws.github.io/eks-charts

```'

# update helm repo
```
helm repo update eks
```
# install alb controller 
```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<your-region> \
  --set vpcId=<your-vpc-id>
```

# Verify that the deployments are running.
```
kubectl get deployment -n kube-system aws-load-balancer-controller
```

# Access the game using DNS name
![alt text](image-1.png)

# Delete the cluster
```
eksctl delete cluster --name demo-cluster --region us-east-1
```