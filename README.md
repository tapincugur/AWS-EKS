# Create EKS CLuster
###If vpc doesn't exist on your cloud account, you can create one.

aws cloudformation deploy --template-file /Users/ugur.tapinc/Desktop/scripts/CloudFormation/twosubnet.yaml --stack-name k8s-vpc-test --region eu-central-1 --profile ugur-playground

#deleting vpc by cloudformation aws cli command 
#aws cloudformation delete-stack --stack-name k8s-vpc-test --region eu-central-1 --profile ugur-playground

##Create EKS Cluster with eksctl ( It takes 15 minutes to be done )
#This 
aws eks --region eu-central-1 --profile ugur-playground create-cluster \
--name eks-test-k8s \
--role-arn arn:aws:iam::318731615233:role/eks-test-k8s-role \
--resources-vpc-config subnetIds=subnet-1,subnet-2,subnet-3,subnet-4,securityGroupIds=sg-0851e8b0ced328e50

#or this
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

#to check cluster status
aws eks --region eu-central-1 --profile ugur-playground describe-cluster --name eks-test-k8s --query cluster.status

# installation for dashboard/metrics of EKS
1. kubernetes dashboard
kubectl apply  -f ~/Desktop/eks/installation/kubernetes-dashboard.yaml
2. heapster: for backend 
kubectl apply -f ~/Desktop/eks/installation/heapster.yaml
3. db with influxd
kubectl apply -f ~/Desktop/eks/installation/influxdb.yaml
4. biding 
kubectl apply -f ~/Desktop/eks/installation/heapster-rbac.yaml
5. Apply the service account
kubectl apply -f ~/Desktop/eks/installation/eks-admin-service-account.yaml
6. bind service 
kubectl apply -f ~/Desktop/eks/installation/heapster-rbac.yaml
7. get token 
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep eks-admin | awk '{print $1}')
copy token to login 
8. start server 
kubectl proxy
9. login 
http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/login

#installation helm 
kubectl create serviceaccount tiller --namespace kube-system
kubectl apply -f ~/Desktop/eks/installation/rbac-config.yaml

#installation prometheus&grafana&alertmanager for collecting metrics and logs of the EKS Cluster ( If you store data, you have to mount disk to your EKS Cluster )

kubectl create namespace prometheus ( You can change type : LoadBalancer to access from Internet )
helm install -f ~/Desktop/eks/installation/prometheus-values.yaml stable/prometheus --name prometheus --namespace prometheus

#If you use small instance type on EKS Cluster Inf., you should increase your instance count for being ready all pods of Prometheus
eksctl scale nodegroup --cluster eks-k8s-test --name eks-k8s-test-nodegroup --nodes 2 --region eu-central-1 --profile ugur-playground

#installation grafana
helm install -f ~/Desktop/eks/installation/grafana-values.yaml stable/grafana --name grafana --namespace grafana
#data source for grafana "url=http://prometheus-server.prometheus.svc.cluster.local "



