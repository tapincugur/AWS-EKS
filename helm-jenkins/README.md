# Jenkins in Kubernetes
Firstly, you should clone this repository on your server.
You can edit "plugin.txt" file according to your code requirements. 

### Build the Docker Image & Push to AWS ECR Service
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

### Deploy Jenkins helm chart to Kubernetes
```bash
# Deploy the Jenkins helm chart
# (same command for install and upgrade)
$ helm upgrade --install jenkins ./helm/jenkins-k8s
```

### Data persistence
By default, in Kubernetes, the Jenkins deployment uses a persistent volume claim that is mounted to `/var/jenkins_home`.
This assures your data is saved across crashes, restarts and upgrades.   

