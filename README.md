# Create-a-VPC-Through-Terraform

## Description.

Terraform is a tool for building infrastructure with various technologies including Amazon AWS, Microsoft Azure, Google Cloud, and vSphere.

Here is a simple document on how to use Terraform to build an AWS VPC along with private/public Subnet and Network Gateway's for the VPC. We will be making 1 VPC with 6 Subnets: 3 Private and 3 Public, 1 NAT Gateways, 1 Internet Gateway, and 2 Route Tables all the creation was automated with appending to your values.

## Features
- Fully Automated (included terraform installation)
- Easy to customise and use as the Terraform modules are created using variables,allowing the module to be customized without altering the module's own source code, and allowing modules to be shared between different configurations.
- Each subnet CIDR block created automatically using cidrsubnet Function (Automated)
- AWS informations are defined using tfvars file and can easily changed (Automated/Manual)
- Project name is appended to the resources that are creating which will make easier to identify the resources.
## Prerequisites for this project
- Need a IAM user access with attached policies for the creation of VPC.
- Knowledge to the working principles of each AWS services especially VPC, EC2 and IP Subnetting.

## Variables used

region - Region of the EC2 (default: us-east-2)

cidr_block - CIDR block for the VPC (default: 10.0.0.0/16)

project - Name of project this VPC is meant for (default: demo)

ami - Amazon Machine Image (AMI) ID

type - Instance type for instance (default: t2.micro)

access_key - access key id for the IAM user

secret_key - secret key for the IAM user

# Terraform Installation and VPC creation code and explanation :
## 1.Creating the provider .tf file
This files contains the provider configuration.
```
provider "aws" {
  region     = var.region
  access_key = var.access_key
  secret_key = var.secret_key
}
```
## 2.Creating the variables.tf file
This file is used to declare the variables we are using in this project.
```
variable "region" {

  description = "Amazon Default Region"    
  default = "ap-south-1"
}

variable "project" {
  default = "zomato"
}

variable "vpc_cidr" { 
  default = "172.16.0.0/16"
```
Lets start creating main.tf file with the details below.

##  3.Creating the main .tf file


