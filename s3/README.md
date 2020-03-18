### Helm Chart Repository using AWS S3
```bash
to create s3 bucket with the bucket policy's like:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowListObjects",
      "Effect": "Allow",
      "Principal": {
        "AWS": ["arn:aws:iam::111144441111:user/testuser"]
      },
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::helm-chart-repo"
    },
    {
      "Sid": "AllowObjectsFetchAndCreate",
      "Effect": "Allow",
      "Principal": {
        "AWS": ["arn:aws:iam::111144441111:user/testuser"]
      },
      "Action": [
        "s3:DeleteObject",
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::helm-chart-repo/*"
    }
  ]
}
```

```bash
Firstly, edit "~/.aws/config" file with the testuser's access&secret keys, and region which you wanna use infrastructure. And then follow these steps:
$ helm plugin install https://github.com/hypnoglow/helm-s3.git 
$ helm S3 init s3://helm-chart-repo/charts
$ helm repo add my-charts s3://helm-chart-repo/charts
$ helm repo list
Example: 
$ helm create application-chart
$ helm package --version 0.0.1 application-chart 
$ helm s3 push application-chart-0.0.1.tgz coolcharts
$ helm fetch coolcharts/application-chart --version "0.0.1"
$ helm install coolcharts/application-chart --version "0.0.1"
$ helm ls
```
