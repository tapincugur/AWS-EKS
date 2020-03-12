# Jenkins in Kubernetes

### Build the Jenkins Docker image
You can build the image yourself
```bash

# Build the image
$ docker build -t  .

# Push the image
$ docker push 
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

