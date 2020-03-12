# Scaling Jenkins on AWS EKS
Firstly, you should clone this repository on your server.
You can edit "plugin.txt" file according to your code requirements. 

### Build the Jenkins Master Docker Image & Push to AWS ECR Service
Build your docker image(jenkins-master)
```bash
$ docker build . -t "jenkins-master" -f  Dockerfile
```
Push the image to AWS ECR Service
```bash
$ eval $( aws ecr get-login --no-include-email --region eu-central-1 --profile ugur-playground | sed 's|https://||' )
$ docker tag jenkins-master:latest ACCOUNT_ID.dkr.ecr.eu-central-1.amazonaws.com/jenkins-master:latest
$ docker push ACCOUNT_ID.dkr.ecr.eu-central-1.amazonaws.com/jenkins-master:latest
```

### Deploy Jenkins helm chart to Kubernetes Cluster
```bash
# Deploy the Jenkins helm chart
$ helm upgrade --install jenkins ./helm/jenkins-k8s --namespace default
```

### Data Persistence
By default, in Kubernetes, the Jenkins deployment uses a persistent volume claim that is mounted to `/var/jenkins_home`.
This assures your data is saved across crashes, restarts and upgrades.

### Login to Jenkins Master
```bash
You can get username&password information in ./jenkins-k8s/values.yaml file
$ URL: kubectl get svc --all-namespaces  | grep jenkins | grep -i loadbalancer|awk '{print $5}'
```

### Build the Jenkins Slave Docker Image & Push to AWS ECR Service
Build your docker image(jenkins-master)
```bash
$ docker build -f  /jnlp-slave/Dockerfile -t "jenkins-slave"
```
Push the image to AWS ECR Service
```bash
$ eval $( aws ecr get-login --no-include-email --region eu-central-1 --profile ugur-playground | sed 's|https://||' )
$ docker tag jenkins-slave:latest ACCOUNT_ID.dkr.ecr.eu-central-1.amazonaws.com/jenkins-slave:latest
$ docker push ACCOUNT_ID.dkr.ecr.eu-central-1.amazonaws.com/jenkins-slave:latest
```
### Configure to Jenkins Kubernetes Plugin for Scaling

Manage Jenkins > Configure System > Cloud Section:
Following steps of the jpeg files (1.jpeg, 2.jpeg, 3.jpeg)

Kubernetes
```bash
$ Name : kubernetes
$ Kubernetes URL: (Run this command on terminal -> "kubectl cluster-info | grep master|awk '{print $6}'" and then test connection, if there is an issue about connection, run this command "kubectl create clusterrolebinding permissive-binding --clusterrole=cluster-admin --user=admin --user=kubelet --group=system:serviceaccounts")
$ Kubernetes Namespace: default
$ Jenkins URL: (Run this command on terminal -> "kubectl get svc --all-namespaces  | grep jenkins | grep -i loadbalancer|awk '{print $4}'")
$ Jenkins tunnel: "(Run this command on terminal -> "kubectl get svc --all-namespaces  | grep k8s-agent | grep -i ClusterIP|awk '{print $4}'"):50000"
```
Pod Labels
```bash
Key: jenkins
Value: slave
```

Pod Template
```bash
Name: jenkins-slave
Labels: jenkins-slave
Usage: "Use this node as much as possible"
```

Container Template
```bash
Name: jenkins-slave
Docker Image: ACCOUNT_ID.dkr.ecr.eu-central-1.amazonaws.com/jenkins-slave:latest
Always pull image	Working directory: /home/jenkins/agent
Command to run: /bin/sh -c
Arguments to pass to the command: cat
```

### Create Two Jobs to Check Scaling
```bash
Create Job 1
Restrict where this project can be run: jenkins-slave
Execute shell: "sleep 120"
Create Job 2 like Job1 
```

### Check Jenkins Slave Pods
```bash
$ kubectl get pods --all-namespaces -o wide  |grep jenkins
```


