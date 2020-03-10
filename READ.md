###If vpc doesn't exist on your cloud account, you can create one.

aws cloudformation deploy --template-file /Users/ugur.tapinc/Desktop/scripts/CloudFormation/twosubnet.yaml  --stack-name ugur-vpc-test --region eu-central-1 --profile ugur-playground

#deleting vpc by cloudformation aws cli command
#aws cloudformation delete-stack --stack-name ugur-vpc-test --region eu-central-1 --profile ugur-playground

##Create EKS Cluster with eksctl ( It takes 15 minutes to be done )

