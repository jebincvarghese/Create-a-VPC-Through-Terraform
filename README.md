# Create-a-VPC-Through-Terraform

### Description.

Terraform is a tool for building infrastructure with various technologies including Amazon AWS, Microsoft Azure, Google Cloud, and vSphere.

Here is a simple document on how to use Terraform to build an AWS VPC along with private/public Subnet and Network Gateway's for the VPC. We will be making 1 VPC with 6 Subnets: 3 Private and 3 Public, 1 NAT Gateways, 1 Internet Gateway, and 2 Route Tables all the creation was automated with appending to your values.

### Features
- Fully Automated (included terraform installation)
- Easy to customise and use as the Terraform modules are created using variables,allowing the module to be customized without altering the module's own source code, and allowing modules to be shared between different configurations.
- Each subnet CIDR block created automatically using cidrsubnet Function (Automated)
- AWS informations are defined using tfvars file and can easily changed (Automated/Manual)
- Project name is appended to the resources that are creating which will make easier to identify the resources.
### Prerequisites for this project
- Need a IAM user access with attached policies for the creation of VPC.
- Knowledge to the working principles of each AWS services especially VPC, EC2 and IP Subnetting.

### Variables used

- region - Region of the EC2 (default: us-east-2)
- cidr_block - CIDR block for the VPC (default: 10.0.0.0/16)
- project - Name of project this VPC is meant for (default: demo)
- ami - Amazon Machine Image (AMI) ID
- type - Instance type for instance (default: t2.micro)
- access_key - access key id for the IAM user
- secret_key - secret key for the IAM user

# Terraform Installation and VPC creation code and explanation :
### 1.Creating the provider.tf file
This files contains the provider configuration.
```
provider "aws" {
  region     = var.region
  access_key = var.access_key
  secret_key = var.secret_key
}
```
### 2.Creating the variables.tf file
This file is used to declare the variables we are using in this project.
```
variable "region" {

  description = "Amazon Default Region"    
  default = "ap-south-1"
}

variable "project" {
  default = "myvpc"
}

variable "vpc_cidr" { 
  default = "172.16.0.0/16"
```
Lets start creating main.tf file with the details below.

##  3.Creating the vpc.tf file

>Fetching Availability Zones Names

```
data "aws_availability_zones" "az" {
    
  state = "available"

}
```

**vpc Creation**
 ```
resource "aws_vpc" "vpc" {
    
  cidr_block           = var.vpc_cidr
  instance_tenancy     = "default"
  enable_dns_support   = true
  enable_dns_hostnames = true  
  tags = {
    Name = "${var.project}-vpc"
    project = var.project
  }
  lifecycle {
    create_before_destroy = true
  }
}
```
> Attaching Internet gateWay
```
resource "aws_internet_gateway" "igw" {
    
  vpc_id = aws_vpc.vpc.id
  tags = {
    Name = "${var.project}-igw"
    project = var.project
  }
    
}
```
> Creating Subnets Public1
```
resource "aws_subnet" "public1" {
    
  vpc_id                   = aws_vpc.vpc.id
  cidr_block               = cidrsubnet(var.vpc_cidr,3,0)                        
  map_public_ip_on_launch  = true
  availability_zone        = data.aws_availability_zones.az.names[0]
  tags = {
    Name = "${var.project}-public1"
    project = var.project
  }
}
```
>Creating Subnets Public2
```
resource "aws_subnet" "public2" {
    
  vpc_id                   = aws_vpc.vpc.id
  cidr_block               = cidrsubnet(var.vpc_cidr,3,1)
  map_public_ip_on_launch  = true
  availability_zone        = data.aws_availability_zones.az.names[1]
  tags = {
    Name = "${var.project}-public2"
    project = var.project
  }
}
```
> Creating Subnets Public3
```
resource "aws_subnet" "public3" {
    
  vpc_id                   = aws_vpc.vpc.id
  cidr_block               = cidrsubnet(var.vpc_cidr,3,2)
  map_public_ip_on_launch  = true
  availability_zone        = data.aws_availability_zones.az.names[2]
  tags = {
    Name = "${var.project}-public3"
    project = var.project
  }
}
```
> Creating Subnets Private1
```
resource "aws_subnet" "private1" {
    
  vpc_id                   = aws_vpc.vpc.id
  cidr_block               = cidrsubnet(var.vpc_cidr,3,3)
  map_public_ip_on_launch  = false
  availability_zone        = data.aws_availability_zones.az.names[0]
  tags = {
    Name = "${var.project}-private1"
    project = var.project
  }
}
```
> Creating Subnets Private2
```
resource "aws_subnet" "private2" {
    
  vpc_id                   = aws_vpc.vpc.id
  cidr_block               = cidrsubnet(var.vpc_cidr,3,4)
  map_public_ip_on_launch  = false
  availability_zone        = data.aws_availability_zones.az.names[1]
  tags = {
    Name = "${var.project}-private2"
    project = var.project
  }
}
```
> Creating Subnets Private3
```
resource "aws_subnet" "private3" {
    
  vpc_id                   = aws_vpc.vpc.id
  cidr_block               = cidrsubnet(var.vpc_cidr,3,5)
  map_public_ip_on_launch  = false
  availability_zone        = data.aws_availability_zones.az.names[2]
  tags = {
    Name = "${var.project}-private3"
    project = var.project
  }
}
```
> **Elatic Ip Allocation**
```
resource "aws_eip" "eip" {
  vpc      = true
  tags     = {
    Name    = "${var.project}-nat-eip"
    project = var.project
  }
}
```
> **Creating Nat GateWay**

```
resource "aws_nat_gateway" "nat" {
    
  allocation_id = aws_eip.eip.id
  subnet_id     = aws_subnet.public2.id

  tags     = {
    Name    = "${var.project}-nat"
    project = var.project
  }

}
```
> **RouteTable Creation public**
```
resource "aws_route_table" "public" {
    
  vpc_id = aws_vpc.vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }

  tags     = {
    Name    = "${var.project}-route-public"
    project = var.project
  }
}
```

> **RouteTable Creation Private**

```
resource "aws_route_table" "private" {
    
  vpc_id = aws_vpc.vpc.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.nat.id
  }

  tags     = {
    Name    = "${var.project}-route-private"
    project = var.project
  }
}
```

> RouteTable Association subnet Public1 route table public
```
resource "aws_route_table_association" "public1" {
  subnet_id      = aws_subnet.public1.id
  route_table_id = aws_route_table.public.id
}
```
> RouteTable Association subnet Public2 route table public
```
resource "aws_route_table_association" "public2" {
  subnet_id      = aws_subnet.public2.id
  route_table_id = aws_route_table.public.id
}
```
> RouteTable Association subnet Public3  route table public
```
resource "aws_route_table_association" "public3" {
  subnet_id      = aws_subnet.public3.id
  route_table_id = aws_route_table.public.id
}
```

> RouteTable Association subnet Private1  route table public
```
resource "aws_route_table_association" "private1" {
  subnet_id      = aws_subnet.private1.id
  route_table_id = aws_route_table.private.id
}
```
> RouteTable Association subnet private2  route table public
```
resource "aws_route_table_association" "private2" {
  subnet_id      = aws_subnet.private2.id
  route_table_id = aws_route_table.private.id
}
```
> RouteTable Association subnet private3  route table public
```
resource "aws_route_table_association" "private3" {
  subnet_id      = aws_subnet.private3.id
  route_table_id = aws_route_table.private.id
}
```
