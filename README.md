# Create EKS CLuster
```bash
If vpc doesn't exist on your cloud account, you can create one(Two Public, Two Private Subnet in VPC).
$ aws cloudformation deploy --template-file ./CloudFormation/twosubnet.yaml  --stack-name k8s-vpc-test --region eu-central-1 --profile ugur-playground

Deleting vpc by cloudformation (aws cli command)
# $ aws cloudformation delete-stack --stack-name k8s-vpc-test --region eu-central-1 --profile ugur-playground
```
### Create EKS Cluster (It takes 15 minutes to be done)
```bash
You can create EKS Cluster with "aws cli" or "eksctl"

aws cli command:
$ aws eks --region eu-central-1 --profile ugur-playground create-cluster \
--name eks-test-k8s \
--role-arn arn:aws:iam::318731615233:role/eks-test-k8s-role \
--resources-vpc-config subnetIds=subnet-1,subnet-2,subnet-3,subnet-4,securityGroupIds=sg-0851e8b0ced328e50

eksctl command:
$ eksctl create cluster \
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

to check EKS cluster status
$ aws eks --region eu-central-1 --profile ugur-playground describe-cluster --name eks-test-k8s --query cluster.status
```

### Kubernetes Dashboard
```bash
1) kubernetes dashboard
$ kubectl apply  -f ~/Desktop/eks/installation/kubernetes-dashboard.yaml
2) Apply the service account
$ kubectl apply -f ~/Desktop/eks/installation/eks-admin-service-account.yaml
3) get token 
$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep eks-admin | awk '{print $1}')
copy token to login 
4) start server 
$ kubectl proxy
5) login 
http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/login
```
### Helm
```bash
$ kubectl create serviceaccount tiller --namespace kube-system 
$ kubectl apply -f ~/Desktop/eks/installation/rbac-config.yaml
```
### Prometheus
```bash
$ kubectl create namespace prometheus #( You can change type : LoadBalancer to access from Internet )
$ helm install -f ~/Desktop/eks/installation/prometheus-values.yaml stable/prometheus --name prometheus --namespace prometheus
```
### Metric-Server
```bash
$ kubectl create -f ./metric-server/
```
### Grafana
```bash
$ helm install -f ~/Desktop/eks/installation/grafana-values.yaml stable/grafana --name grafana --namespace grafana 

Note: I share with you some grafana dashboards which you can use for monitoring your env., are in dashboards folder.
```
### ALB Ingress Controller
```bash

Firstly, you should add the tag on private and public subnets for subnet auto discovery. to allow ingress controller auto discover subnets used for ALBs 

Private:
$ Key: kubernetes.io/role/internal-elb Value: "Empty or just write 1"
Public:
$ Key: kubernetes.io/role/elb: "Empty or just write 1"

Go to IAM console and create ingress controller policy, and attach that policy to EKS Cluster Worker Nodes Role
I share with you policy file "./ingress-policy.json"

Create an IAM OIDC provider and associate it with your cluster
$ eksctl utils associate-iam-oidc-provider --cluster=eks-k8s-test --approve --region eu-central-1 --profile ugur-playground

Deploy RBAC Roles and RoleBindings needed by the AWS ALB Ingress controller
$ kubectl apply -f ~/Desktop/eks/installation/ingress/rbac-role.yaml

Deploy the AWS ALB Ingress controller YAML ( Edit line "- --cluster-name=EKS_CLUSTER_NAME" Your EKS Cluster Name )
$ kubectl apply -f ~/Desktop/eks/installation/ingress/alb-ingress-controller.yaml

Verify that the deployment was successful and the controller started 
$ kubectl get pods -n kube-system | grep -i "alb-ingress-controller" | awk '{print $1}'

Also, Shared with you ingress file examples "./ingress/ingress.yaml"
```
### Application running on EKS CLuster (2048-game)
```bash
$ kubectl apply -f ~/Desktop/eks/installation/2048/a1.com/2048-namespace.yaml
$ kubectl apply -f ~/Desktop/eks/installation/2048/a1.com/2048-deployment.yaml
$ kubectl apply -f ~/Desktop/eks/installation/2048/a1.com/2048-service.yaml
$ kubectl apply -f ~/Desktop/eks/installation/2048/a2.com/2048-deployment.yaml
$ kubectl apply -f ~/Desktop/eks/installation/2048/a2.com/2048-service.yaml
$ kubectl apply -f ~/Desktop/eks/installation/ingress/ingress.yaml

Ping that url: "kubectl get ingress  --all-namespaces   |grep 2048-game |awk '{print $4}'"
and edit your local hosts file like:
sudo vi /etc/hosts
1.1.1.1 a1.com a2.com
ingress routes traffic a1.com to "service-2048" service , and a2.com traffic to "service-2048-2" service.
```
