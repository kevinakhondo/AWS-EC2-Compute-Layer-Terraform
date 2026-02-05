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
variable "vpc_id" {
  description = "VPC ID where EC2 will be deployed"
  type        = string
}

```
### Step 3: Create Main.tf

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

### Step 4: Create Security Group (EC2 Firewall)

In _main.tf_, add the following

```
resource "aws_security_group" "ec2_sg" {
  name        = "${var.project_name}-ec2-sg"
  description = "Allow SSH and outbound traffic"
  vpc_id      = var.vpc_id

  ingress {
    description = "SSH access"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["YOUR_PUBLIC_IP/32"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Project = var.project_name
  }
}

```

Remember to replace YOUR_PUBLIC_IP. The easiest way is to Google.

### Step 5:  Create IAM Instance Profile

For EC2 to attach a role, a profile is needed.

```
resource "aws_iam_instance_profile" "ec2_profile" {
  name = "${var.project_name}-ec2-profile"
  role = var.ec2_role_name
}

```
### Step 6: Create EC2 Instance

```
resource "aws_instance" "data_ec2" {
  ami                    = "ami-0c02fb55956c7d316" # Amazon Linux 2 (us-east-1)
  instance_type          = "t3.micro"
  subnet_id              = var.public_subnet_id
  iam_instance_profile   = aws_iam_instance_profile.ec2_profile.name
  vpc_security_group_ids = [aws_security_group.ec2_sg.id]

  tags = {
    Name    = "${var.project_name}-ec2"
    Project = var.project_name
  }

  user_data = file("user_data.sh")
}

```
### Step 7: Create user_data.sh
This enables EC2 to connect directly to S3

```
#!/bin/bash
yum update -y
yum install -y awscli

```
### Step 8: Create outputs.tf

```
output "ec2_instance_id" {
  value = aws_instance.data_ec2.id
}

output "ec2_public_ip" {
  value = aws_instance.data_ec2.public_ip
}

```
### Step9: Connect Modules

Now, go to the root layer, in this case is:

```
cd aws-data-engineering
```
This is typically the folder containing the subfolder modules.
In it, create _main.tf_ if it doesn't exist and add:

```
module "vpc" {
  source       = "./module-03-vpc-terraform"
  vpc_cidr     = var.vpc_cidr
  project_name = var.project_name
}

module "s3" {
  source       = "./module-04-s3-terraform"
  project_name = var.project_name
}

module "iam" {
  source          = "./module-02-iam-terraform"
  raw_bucket_name = module.s3.raw_bucket_name
  project_name    = var.project_name
}

module "ec2" {
  source               = "./module-05-ec2-terraform"
  vpc_id               = module.vpc.vpc_id
  public_subnet_id     = module.vpc.public_subnet_id
  iam_instance_profile = module.iam.ec2_instance_profile
  project_name         = var.project_name
}

```

Add variables.tf

```
variable "project_name" {
  description = "Name of the project"
  type        = string
}

variable "vpc_cidr" {
  description = "CIDR block for the VPC"
  type        = string
}

```

Add terraform.tfvars

```
project_name = "aws-data-engineering-kevins"
vpc_cidr     = "10.0.0.0/16"

```



### Step 10: Run Terraform

Start by Validating

```
terraform validate
```
By running that, you are likely to run into errors. Let's see how you debug. I've debugged mine.

Note that I am new to terraform. Also, vpc is not found in s3.


