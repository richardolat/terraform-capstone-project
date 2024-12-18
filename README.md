![image](https://github.com/user-attachments/assets/9eed0c33-ba74-4a15-83b2-5f59d0472d1f)![image](https://github.com/user-attachments/assets/5afb9e6b-a94f-4c6f-9a38-abb976f12108)# terraform-capstone-project
# The project involves designing and implementing a scalable, secure, and cost-effective WordPress solution using various AWS services. It specifically uses automation through Terraform to achieving a streamlined and reproducible deployment process.

##  VPC Setup

#### Create a VPC with one Public Subnet for resources like the NAT Gateway and Load Balancer and one Private Subnet for EC2 instances to host the WordPress application.

#### Create the main project directory: *mkdir wordpress-terraform**
![image](https://github.com/user-attachments/assets/d464d1c5-3d63-4290-b975-9192d6777807)



#### Create subdirectories for modular files: *mkdir vpc rds efs alb asg**
![image](https://github.com/user-attachments/assets/608a942e-ee5d-440d-a49e-62540f9c94d4)


#### Create the required files in their respective directories: *touch main.tf variables.tf outputs.tf*  - *touch vpc/vpc.tf vpc/subnets.tf vpc/nat_gateway.tf*  - *touch rds/rds.tf*  *touch efs/efs.tf*  - *touch alb/alb.tf*  - *touch asg/asg.tf* - *touch terraform.tfvars*
![image](https://github.com/user-attachments/assets/3c5d9a4f-7f00-4576-a8c7-1587aff5e19b)


## Write the Terraform Configuration

#### The main.tf file ties the project together by including all the individual modules. Add the following code to main.tf:
** # Provider Configuration
provider "aws" {
  region = var.region
}

# VPC Module
module "vpc" {
  source = "./vpc"
}

# RDS Module
module "rds" {
  source = "./rds"
}

# EFS Module
module "efs" {
  source = "./efs"
}

# ALB Module
module "alb" {
  source = "./alb"
}

# Auto Scaling Module
module "asg" {
  source = "./asg"
} **
![image](https://github.com/user-attachments/assets/19e79c15-630f-49fa-8162-ab107159f96b)



#### The variable.tf defines variables for customization. Add the following code to variables.tf:
** # AWS Region
variable "region" {
  default = "us-east-1"
}

# VPC and Subnet CIDRs
variable "vpc_cidr_block" {
  default = "10.0.0.0/16"
}

variable "public_subnet_cidr" {
  default = "10.0.1.0/24"
}

variable "private_subnet_cidr" {
  default = "10.0.2.0/24"
}

# RDS Configuration
variable "db_username" {
  default = "wordpress"
}

variable "db_password" {
  default = "securepassword"
}

# Auto Scaling Configuration
variable "instance_type" {
  default = "t2.micro"
} **
![image](https://github.com/user-attachments/assets/e4868de7-4dab-4ecb-81c3-162cbc70a6c5)


#### The terraform.tfvars provide values for the variables (It is good practice to use secure, unique values for production). Add the following to terraform.tfvars:
region           = "us-east-1"
db_username      = "admin"
db_password      = "mysecurepassword123"
instance_type    = "t2.micro"
![image](https://github.com/user-attachments/assets/445418a4-ae86-4b82-bb05-ef7c18e3a8cb)


#### The outputs.tf defines outputs to retrieve important information after deployment. Add the following to outputs.tf:
output "vpc_id" {
  value = module.vpc.vpc_id
}

output "alb_dns_name" {
  value = module.alb.alb_dns_name
}

output "rds_endpoint" {
  value = module.rds.rds_endpoint
}
![image](https://github.com/user-attachments/assets/14101e52-e4d0-4a0b-9ef3-6fd143ff5ac5)


## Write the Module Files

#### The vpc.tf: It creates a Virtual Private Cloud (VPC) for the project, providing an isolated network for all resources. It also defines the CIDR block for the IP address range.

resource "aws_vpc" "wordpress_vpc" {
  cidr_block           = var.vpc_cidr_block
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "wordpress-vpc"
  }
}
![image](https://github.com/user-attachments/assets/a9003ca4-78b7-4fa6-b951-5baf54588556)



#### subnets.tf: It creates a public subnet for internet-accessible resources (NAT Gateway, Load Balancer) and also creates a private subnet for backend resources (EC2 instances, RDS) with no direct internet access. 
resource "aws_subnet" "public_subnet" {
  vpc_id                  = aws_vpc.wordpress_vpc.id
  cidr_block              = var.public_subnet_cidr
  map_public_ip_on_launch = true
}

resource "aws_subnet" "private_subnet" {
  vpc_id     = aws_vpc.wordpress_vpc.id
  cidr_block = var.private_subnet_cidr
}
![image](https://github.com/user-attachments/assets/622dec00-8bd4-4fab-9b5b-9d45b1064047)

#### nat_gateway.tf creates an Internet Gateway to enable internet access for public subnet resources. It also sets up a NAT Gateway to allow private subnet resources ( EC2) to connect to the internet securely for updates or external communication.
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.wordpress_vpc.id
}

resource "aws_eip" "nat_eip" {
  vpc = true
}

resource "aws_nat_gateway" "nat_gateway" {
  allocation_id = aws_eip.nat_eip.id
  subnet_id     = aws_subnet.public_subnet.id
}
![image](https://github.com/user-attachments/assets/ec7e6a3c-dc4c-4745-8317-1445a61a956b)


## RDS Module
#### rds.tf deploys a MySQL RDS instance to store WordPress data (database backend), Configures database username, password, and storage size and ensures the database is deployed in private subnets for security.
resource "aws_db_instance" "wordpress_rds" {
  allocated_storage    = 20
  engine               = "mysql"
  instance_class       = "db.t3.micro"
  name                 = "wordpressdb"
  username             = var.db_username
  password             = var.db_password
  publicly_accessible  = false
}
![image](https://github.com/user-attachments/assets/117efdb3-fda7-4fd1-b865-c6fc27c55d10)

## EFS Module
#### efs.tf creates an Elastic File System (EFS) for scalable, shared storage and stores WordPress files (uploads, themes) that need to be accessible from multiple EC2 instances in the Auto Scaling group.
resource "aws_efs_file_system" "wordpress_efs" {
  creation_token = "wordpress-efs"
}
![image](https://github.com/user-attachments/assets/6d847360-dca9-466c-9ca1-4b6692a1dd00)

## ALB Module
#### alb.tf deploys an Application Load Balancer (ALB) to distribute incoming traffic across multiple EC2 instances, configures a target group to manage the backend EC2 instances hosting WordPress and ensures high availability and fault tolerance by routing traffic to healthy instances.
resource "aws_lb" "wordpress_alb" {
  name               = "wordpress-alb"
  internal           = false
  load_balancer_type = "application"
}

resource "aws_lb_target_group" "wordpress_tg" {
  name     = "wordpress-target-group"
  port     = 80
  protocol = "HTTP"
}
![image](https://github.com/user-attachments/assets/4f0bbb3f-0de5-4f9f-9065-89229d1e5beb)

## Auto Scaling Module
#### asg.tf configures an Auto Scaling Group (ASG) to manage a fleet of EC2 instances, automatically adjusts the number of instances based on traffic (CPU utilization or incoming traffic load), and ensures WordPress remains highly available during traffic spikes.
resource "aws_autoscaling_group" "wordpress_asg" {
  max_size = 3
  min_size = 1
  desired_capacity = 2
}
![image](https://github.com/user-attachments/assets/a7d7c604-f255-4f93-9f0d-ef2f0c8de735)

## Run the Terraform Scripts


#### cd wordpress-terraform
![image](https://github.com/user-attachments/assets/17a14efd-2e93-499b-90a7-3fb8f259fc28)

#### Initialize Terraform: *terraform init**































































































































































