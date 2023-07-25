# AWS CLI - Aslan 01/10/2023

## References

### <https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html>

### <https://awscli.amazonaws.com/v2/documentation/api/latest/index.html>

### <https://aws.amazon.com/blogs/compute/query-for-the-latest-amazon-linux-ami-ids-using-aws-systems-manager-parameter-store/>

### <https://aws.amazon.com/cloudformation/resources/templates/>

## Installation

## Win, Mac and Linux

<https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html>

## Mac installation with screenshots

<https://graspingtech.com/install-and-configure-aws-cli/>

``` bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip  #install "unzip" if not installed
sudo ./aws/install
```

## Configuration

```bash
aws configure

cat .aws/config
cat .aws/credentials

aws configure --profile cdprojectred_aslan

aws s3 ls --profile cdprojectred_aslan

export AWS_PROFILE=cdprojectred_aslan
export AWS_PROFILE=default

aws configure list-profiles

aws sts get-caller-identity
```

## Key-pair

```text
aws ec2 create-key-pair --key-name <key_name> --query 'KeyMaterial' --output text > <key_name>.pem

aws ec2 describe-key-pairs

aws ec2 delete-key-pair --key-name <key-name>
```

## IAM

```text
aws iam list-users

aws iam create-user --user-name aws-cli-user

aws iam delete-user --user-name aws-cli-user
```

## S3

```text
aws s3 ls

aws s3 mb s3://aslan-cli-bucket

aws s3 cp in-class.yaml s3://aslan-cli-bucket

aws s3 ls s3://aslan-cli-bucket

aws s3 rm s3://aslan-cli-bucket/in-class.yaml

aws s3 rb s3://aslan-cli-bucket
```

## EC2

```bash
aws ec2 describe-instances

aws ec2 run-instances \
   --image-id ami-033b95fb8079dc481 \
   --count 1 \
   --instance-type t2.micro \
   --key-name KEY_NAME_HERE # put your key name

aws ec2 create-tags \
   --resources <INSTANCE_ID_HERE>
   --tags Key=Name, Value="created_from_aws_cli"

aws ec2 describe-instances \
   --filters "Name = key-name, Values = KEY_NAME_HERE" # put your key name

aws ec2 describe-instances --query "Reservations[].Instances[].PublicIpAddress[]"

aws ec2 describe-instances \
   --filters "Name = key-name, Values = KEY_NAME_HERE" --query "Reservations[].Instances[].PublicIpAddress[]" # put your key name

aws ec2 describe-instances \
   --filters "Name = tag-value, Values = TAG_NAME_HERE " --query "Reservations[].Instances[].PublicIpAddress[]" 

aws ec2 describe-instances \
   --filters "Name = instance-type, Values = t2.micro" --query "Reservations[].Instances[].InstanceId[]"

aws ec2 stop-instances --instance-ids INSTANCE_ID_HERE # put your instance id

aws ec2 terminate-instances --instance-ids INSTANCE_ID_HERE # put your instance id

# Working with the latest Amazon Linux AMI

aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --region us-east-1

aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --query 'Parameters[0].[Value]' --output text

aws ec2 run-instances \
   --image-id $(aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --query 
'Parameters[0].[Value]' --output text) \
   --count 1 \
   --instance-type t2.micro

# Update AWS CLI Version 1 on Amazon Linux (comes default) to Version 2

# Remove AWS CLI Version 1
sudo yum remove awscli -y # pip uninstall awscli/pip3 uninstall awscli might also work depending on the image

# Install AWS CLI Version 2
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip  #install "unzip" if not installed
sudo ./aws/install

# Update the path accordingly if needed
export PATH=$PATH:/usr/local/bin/aws
```
