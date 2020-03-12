# Scaling Jenkins in AWS EKS
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

Manage Jenkins > Configure System > Cloud Section









