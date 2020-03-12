# Create EKS CLuster
```bash
If vpc doesn't exist on your cloud account, you can create one.
$ aws cloudformation deploy --template-file /Users/ugur.tapinc/Desktop/scripts/CloudFormation/twosubnet.yaml --stack-name k8s-vpc-test --region eu-central-1 --profile ugur-playground

Deleting vpc by cloudformation (aws cli command)
# $ aws cloudformation delete-stack --stack-name k8s-vpc-test --region eu-central-1 --profile ugur-playground
```
## Create EKS Cluster with aws-cli&eksctl (It takes 15 minutes to be done)
```bash
This 
$ aws eks --region eu-central-1 --profile ugur-playground create-cluster \
--name eks-test-k8s \
--role-arn arn:aws:iam::318731615233:role/eks-test-k8s-role \
--resources-vpc-config subnetIds=subnet-1,subnet-2,subnet-3,subnet-4,securityGroupIds=sg-0851e8b0ced328e50

#or this \
eksctl create cluster \
--name eks-k8s-test \
--version 1.14 \
--nodegroup-name eks-k8s-test-nodegroup \
--node-type t3.small \
--node-ami auto \
--vpc-private-subnets=subnet-1,subnet-2 \
--vpc-public-subnets=subnet-1,subnet-2 \
--node-private-networking \
--nodes 1 \
--nodes-min 1 \
--nodes-max 4 \
--alb-ingress-access \
--region eu-central-1 \
--profile ugur-playground

#to check cluster status \
aws eks --region eu-central-1 --profile ugur-playground describe-cluster --name eks-test-k8s --query cluster.status
```
# Kubernetes Dashboard 
1) kubernetes dashboard
kubectl apply  -f ~/Desktop/eks/installation/kubernetes-dashboard.yaml
2) Apply the service account
kubectl apply -f ~/Desktop/eks/installation/eks-admin-service-account.yaml
3) get token 
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep eks-admin | awk '{print $1}')
copy token to login 
4) start server 
kubectl proxy
5) login 
http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/login

# Helm
kubectl create serviceaccount tiller --namespace kube-system \
kubectl apply -f ~/Desktop/eks/installation/rbac-config.yaml

# Prometheus
kubectl create namespace prometheus #( You can change type : LoadBalancer to access from Internet )\
helm install -f ~/Desktop/eks/installation/prometheus-values.yaml stable/prometheus --name prometheus --namespace prometheus

# Grafana 
helm install -f ~/Desktop/eks/installation/grafana-values.yaml stable/grafana --name grafana --namespace grafana 

#I share with you some grafana dashboards which you can use for monitoring your env., are in dashboards folder.

# ALB Ingress Controller

#Create an IAM OIDC provider and associate it with your cluster \
eksctl utils associate-iam-oidc-provider --cluster=eks-k8s-test --approve --region eu-central-1 --profile ugur-playground

#Deploy RBAC Roles and RoleBindings needed by the AWS ALB Ingress controller \
kubectl apply -f ~/Desktop/eks/installation/ingress/rbac-role.yaml

#Deploy the AWS ALB Ingress controller YAML \
kubectl apply -f ~/Desktop/eks/installation/ingress/alb-ingress-controller.yaml

#Verify that the deployment was successful and the controller started \
kubectl get pods -n kube-system | grep -i "alb-ingress-controller" | awk '{print $1}'

# Application running on EKS CLuster (2048-game)
kubectl apply -f ~/Desktop/eks/installation/2048/2048-namespace.yaml \
kubectl apply -f ~/Desktop/eks/installation/2048/2048-deployment.yaml \
kubectl apply -f ~/Desktop/eks/installation/2048/2048-service.yaml
