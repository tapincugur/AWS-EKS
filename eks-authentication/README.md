### EKS CLuster Authentication Config
```bash
Edit configmap/aws-auth file to add iam users/roles.
You can see an examples file "auth.yaml" about how to change configmap/aws-auth file.
$ kubectl edit -n kube-system configmap/aws-auth 
```