### EKS CLuster Authentication Config
Edit configmap/aws-auth file to add iam users/roles.
```bash
You can see an examples file "auth.yaml" about how to change configmap/aws-auth.
$ kubectl edit -n kube-system configmap/aws-auth 
```