## About

- Using the CloudFormation templates, you can provision the following:
  - A bucket (say, in AWS region `ap-northeast-1` aka Tokyo) that will be the 'destination' in S3-CRR
  - Another bucket (say, in AWS region `ap-south-1` aka Mumbai) that will be the 'source' in S3-CRR
  - A (short-lived) fleet of EC2 spot instances that generate and put a number of "large" files into the source bucket
  
- There are good reasons why these are separate stacks (and templates), and must be provisioned in a specific order:
  - The bucket to replicate to (aka 'destination') will be provisioned first in some AWS region (the first stack), say `ap-northeast-1`.
  - The bucket to replicate from (aka 'source') will be provisioned afterwards, in a different AWS region (the second stack), say `ap-south-1`, and presumes that the 'destination' bucket exists.
  - A short-lived fleet of EC2 instances to generate some files and put the same into the S3 bucket will be provisioned in the same region as the 'source' bucket, say `ap-south-1`, and presumes that the 'source' bucket exists.
  
### Current limitations

- The fleet of EC2 instances currently only require access to an S3 API endpoint. In the simplest case, this is achieved by launching the fleet in a public subnet and associating public IP addresses. The (arguably better) alternative is to launch the fleet within a private subnet with no special outbound access, and have a VPC endpoint for S3 provisioned and configured for the private subnet.
- The template currently assumes (parameter) defaults for the EC2 instance fleet at 20 spot instances that each generate 10 files of 2 GB each for a total of 400 GB put into the S3 bucket. You may provision any number of such stacks, or alternatively tweak the CloudFormation templates for these numbers.
  
## Steps

### General instructions (for CloudFormation)

General instructions to provision CloudFormation stacks:

- Given a template, you can provision a CloudFormation stack as follows:

```bash

$ aws --region ap-south-1 cloudformation create-stack \
    --stack-name my-stack \
    --template-body file://template.yaml \
    --parameters \
    ParameterKey=foo,ParameterValue=bar \

```

- If a stack is creating or modifying IAM resources or permissions, you may need to acknowledge it by supplying capabilities, by adding the following argument to the ```create-stack``` call

``` --capabilities CAPABILITY_IAM ```

- To monitor a stack, you may use:

```bash

$ aws --region ap-south-1 cloudformation describe-stacks \
    --stack-name my-stack

$ aws --region ap-south-1 cloudformation describe-stack-events \
    --stack-name my-stack
    
```
- You can also use ```describe-stacks``` call to print the output values from a stack, which may be required to provision other stacks

- To delete a stack, you may use:

```bash

$ aws --region ap-south-1 cloudformation delete-stack \
    --stack-name my-stack

```

### Specific instructions

The [stacks](stacks) folder consists of three other folders that each contain
  - a CloudFormation template: `template.yaml`
  - an example file that indicates the parameters required to create a stack using the template: `parameters.example.json`
  - a `metadata.json` (non-standard) that describes some aspects of the stack that may be provisioned, such as whether capabilites are required to be acknowledged, etc.

The folders have been numbered as a hint to the sequence in which the stacks need to be provisioned. Look at the `parameters.example.json` to learn about the parameters which must be supplied when creating the stacks. Defaults have been provided within the templates for some values, but you must substitute the values specific to your environment within the chosen AWS account for other parameters.

- Step 1: Provision the bucket to replicate to (say, in region `ap-northeast-1`)

Template and example parameters are in [01-replicate-to-bucket](stacks/01-replicate-to-bucket)

- Step 2: Provision the bucket to replicate from (say, in region `ap-south-1`)

Template and example parameters are in [02-replicate-from-bucket](stacks/02-replicate-from-bucket)

- Step 3: Provision the EC2 fleet to generate files and put them into the 'source' S3 bucket (typically, to the same region as the 'source' bucket)

Template and example parameters are in [03-put-files-in-s3](stacks/03-put-files-in-s3)

- Step 4: Monitoring and cleanup

Provided that the spot bid is successful, the desired number of EC2 instances will be provisioned and they may take several minutes to generate, and then put files into the 'source' s3 bucket. This should not take more than 30 minutes overall, but YMMV.

```bash
# Inspect the source bucket for files copied in by the 'put-files-in-s3' EC2 instance fleet

$ aws --region ap-south-1 s3 ls s3://my-source-bucket/files/

```
You can delete the stack that was provisioned to create a fleet of EC2 instances to generate the files.

- Step 5: Monitor the progress of S3 CRR.

[S3 Inventory](http://docs.aws.amazon.com/AmazonS3/latest/dev/storage-inventory.html) is turned on for the 'destination' S3 bucket, so you can look at the inventory files when generated to verify which files replicated to the bucket and when.

### Costs incurred

- EC2 instances run for a short duration (typically 30 minutes) at the spot price
- S3 CRR data transfer
- S3 storage
