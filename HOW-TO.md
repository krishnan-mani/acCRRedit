About
====

- Using the CloudFormation templates, you can provision the following:
  - A bucket (say, in AWS region 'ap-northeast-1' aka Tokyo) that will be the 'destination' in S3-CRR
  - Another bucket (say, in AWS region 'ap-south-1' aka Mumbai) that will be the 'source' in S3-CRR
  - A stack consisting of a short-lived fleet of EC2 instances that generate and put a number of "large" files into the source bucket
  
Steps
====

General instructions to provision CloudFormation stacks:

- Given a template, you can provision a CloudFormation stack as follows:

```bash

$ aws --region ap-south-1 cloudformation create-stack \
    --stack-name my-stack \
    --template-body file://template.yaml \
    --parameters \
    ParameterKey=foo,ParameterValue=bar \
    --capabilities CAPABILITY_IAM

```

- To monitor a stack, you may use the following

```bash

$ aws --region ap-south-1 cloudformation describe-stacks \
    --stack-name my-stack

$ aws --region ap-south-1 cloudformation describe-stack-events \
    --stack-name my-stack
    
```

- To delete a stack, you may use something like

```bash

$ aws --region ap-south-1 cloudformation delete-stack \
    --stack-name my-stack

```


- Step 1: Provision the bucket to replicate to

Template and example parameters are in [01-replicate-to-bucket](stacks/01-replicate-to-bucket)

- Step 2: Provision the bucket to replicate from

Template and example parameters are in [02-replicate-from-bucket](stacks/02-replicate-from-bucket)

- Step 3: Provision the EC2 fleet to generate files and put them into the 'source' S3 bucket

Template and example parameters are in [02-replicate-from-bucket](stacks/02-replicate-from-bucket)

- Step 4: Cleanup

You can delete the stack that was provisioned to create a fleet of EC2 instances to generate the files

Step 5: Monitor the progress of S3 CRR.


Costs incurred
====

- Run a fleet of EC2 instances for a short duration (typically 30 minutes)
- S3 CRR data transfer
- S3 storage