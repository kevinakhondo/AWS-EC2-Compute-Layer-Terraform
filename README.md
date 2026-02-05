# AWS-EC2-Compute-Layer-Terraform
Launch an EC2 Instance Terraform
## Introduction
The goal is to create an EC2 Instance that:
- Lives in your public subnet
- Uses an IAM role (no access keys)
- Can read/write to S3
- can connect to RDS later

The ideal infra is:

```
EC2
 ├── Subnet: public
 ├── IAM Role: ec2_data_role
 ├── Security Group: controlled access
 └── Data access: S3 via IAM (no secrets)

```
The structure of the folder is

```
aws-data-engineering/module-05-ec2-terraform/
├── main.tf
├── variables.tf
├── outputs.tf
└── user_data.sh

```

## Hands On
### Step 1: Create Folder

In your terminal, create the following folder

```
cd aws-data-engineering
mkdir -p module-05-ec2-terraform
cd module-05-ec2-terraform
```


