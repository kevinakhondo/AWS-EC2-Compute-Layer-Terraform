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
