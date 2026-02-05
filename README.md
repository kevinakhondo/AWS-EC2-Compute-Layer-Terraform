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
### Step 2: Create variables.tf
By creating this, we are connecting the EC2 to earlier modules we've done on the previous resipositories.
In your VS, open the created folder and create _variables.tf_ file.
Add the following:

```
variable "aws_region" {
  type    = string
  default = "us-east-1"
}

variable "project_name" {
  type = string
}

variable "public_subnet_id" {
  description = "Public subnet ID from VPC module"
  type        = string
}

variable "ec2_role_name" {
  description = "IAM role name for EC2"
  type        = string
}

```
### Step 3: Create Maint.tf

```
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

```







