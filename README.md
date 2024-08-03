# AWS Lift and Shift Project 

## Overview

This project involves migrating a multi-tier web application stack (vProfile) from a local data center to the AWS cloud using a lift-and-shift strategy. The key AWS services used include EC2 instances, Elastic Load Balancer, Auto Scaling, S3, Route 53, IAM, ACM, and EBS.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Project Setup](#project-setup)
   - [1. Launch EC2 Instances](#1-launch-ec2-instances)
   - [2. Build and Deploy Artifacts](#2-build-and-deploy-artifacts)
   - [3. Set Up Load Balancer and DNS](#3-set-up-load-balancer-and-dns)
   - [4. Set Up Auto Scaling Group](#4-set-up-auto-scaling-group)
3. [Validation and Summary](#validation-and-summary)
4. [Future Improvements](#future-improvements)

## Prerequisites

- An AWS account
- Basic knowledge of AWS services
- AWS CLI installed and configured
- SSH client (e.g., PuTTY for Windows or Terminal for macOS/Linux)

## Project Setup

### 1. Launch EC2 Instances

1. **Launch EC2 Instances**:
   - **Via AWS Console**:
     1. Open the EC2 Management Console.
     2. Click `Launch Instance`.
     3. Choose an Amazon Machine Image (AMI).
     4. Select `t2.micro` (free tier) as the instance type.
     5. Configure instance details (number of instances, network settings).
     6. Add storage and tags.
     7. Configure a security group (e.g., `vprofile app security group`).
     8. Review and launch the instance.
     9. Select or create a key pair (e.g., `vprofile-prod-key`).
     10. Click `Launch`.

2. **Connect to EC2 Instance via SSH**:
   - **Command**:
     ```bash
     ssh -i "vprofile-prod-key.pem" ec2-user@<instance-public-dns>
     ```
   - **Explanation**:
     - `ssh`: Secure Shell command to connect to remote servers.
     - `-i "vprofile-prod-key.pem"`: Specifies the private key file for authentication.
     - `ec2-user@<instance-public-dns>`: The default user and public DNS of the EC2 instance.

3. **Configure Instances**:
   - **Update Package List**:
     ```bash
     sudo yum update -y
     ```
   - **Install Software** (e.g., Tomcat, RabbitMQ, Memcached, MySQL):
     ```bash
     sudo yum install -y tomcat rabbitmq-server memcached mysql-server
     ```
   - **Start and Enable Services**:
     ```bash
     sudo systemctl start tomcat
     sudo systemctl enable tomcat
     sudo systemctl start rabbitmq-server
     sudo systemctl enable rabbitmq-server
     sudo systemctl start memcached
     sudo systemctl enable memcached
     sudo systemctl start mysqld
     sudo systemctl enable mysqld
     ```
   - **Check Status of Services**:
     ```bash
     sudo systemctl status tomcat
     sudo systemctl status rabbitmq-server
     sudo systemctl status memcached
     sudo systemctl status mysqld
     ```
   - **Explanation**:
     - `sudo yum update -y`: Updates all installed packages to the latest version.
     - `sudo yum install -y <package>`: Installs specified packages.
     - `sudo systemctl start <service>`: Starts the specified service.
     - `sudo systemctl enable <service>`: Enables the service to start on boot.
     - `sudo systemctl status <service>`: Checks the status of the service.

4. **Verify Security Group Rules**:
   - **Command**:
     ```bash
     aws ec2 describe-security-groups --group-ids <security-group-id>
     ```
   - **Explanation**:
     - `aws ec2 describe-security-groups`: Describes the security group settings.
     - `--group-ids <security-group-id>`: Specifies the security group ID.

### 2. Build and Deploy Artifacts

1. **Create S3 Bucket and IAM User**:
   - **Create S3 Bucket**:
     - **Command**:
       ```bash
       aws s3 mb s3://<bucket-name>
       ```
     - **Explanation**:
       - `aws s3 mb`: Creates a new bucket.
       - `s3://<bucket-name>`: Specifies the bucket name.

   - **Create IAM User**:
     - **Command**:
       ```bash
       aws iam create-user --user-name <user-name>
       aws iam attach-user-policy --user-name <user-name> --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
       ```
     - **Explanation**:
       - `aws iam create-user`: Creates a new IAM user.
       - `aws iam attach-user-policy`: Attaches a policy to the IAM user.

2. **Configure AWS CLI**:
   - **Command**:
     ```bash
     aws configure
     ```
   - **Explanation**:
     - Prompts for AWS Access Key ID, Secret Access Key, default region name, and output format.

3. **Build Application Artifact**:
   - **Command**:
     ```bash
     mvn clean package
     ```
   - **Explanation**:
     - `mvn clean package`: Uses Maven to build the application artifact.

4. **Deploy Artifact to EC2**:
   - **Upload Artifact**:
     ```bash
     aws s3 cp <artifact-file> s3://<bucket-name>/
     ```
   - **Download and Deploy on EC2**:
     ```bash
     aws s3 cp s3://<bucket-name>/<artifact-file> /path/to/deploy/
     cd /path/to/deploy/
     sudo tar -xzvf <artifact-file>
     ```
   - **Explanation**:
     - `aws s3 cp`: Copies files to/from S3.
     - `tar -xzvf`: Extracts a tarball file.

### 3. Set Up Load Balancer and DNS

1. **Create Application Load Balancer**:
   - **Via AWS Console**:
     1. Open the EC2 Management Console.
     2. Navigate to `Load Balancers` and click `Create Load Balancer`.
     3. Choose `Application Load Balancer`.
     4. Configure listeners (HTTP/HTTPS) and security groups.
     5. Define target groups and health checks.

2. **Configure Route 53**:
   - **Create Private DNS Zone**:
     - **Via AWS Console**:
       1. Open the Route 53 Management Console.
       2. Click `Create Hosted Zone`.
       3. Choose `Private Hosted Zone` and specify domain details.

   - **Add CNAME Record**:
     - **Via AWS Console**:
       1. Navigate to the Hosted Zone.
       2. Click `Create Record Set`.
       3. Select `CNAME` and enter the load balancer endpoint.

3. **Verify Load Balancer**:
   - **Command**:
     ```bash
     curl -I https://<load-balancer-url>
     ```
   - **Explanation**:
     - `curl -I`: Fetches HTTP headers to verify the response from the load balancer.

### 4. Set Up Auto Scaling Group

1. **Create AMI**:
   - **Command**:
     ```bash
     aws ec2 create-image --instance-id <instance-id> --name "vprofile-app-image" --no-reboot
     ```
   - **Explanation**:
     - `aws ec2 create-image`: Creates an AMI from the instance.
     - `--instance-id <instance-id>`: Specifies the instance ID.
     - `--name "vprofile-app-image"`: Names the AMI.
     - `--no-reboot`: Creates the AMI without rebooting the instance.

2. **Create Launch Template**:
   - **Via AWS Console**:
     1. Open the EC2 Management Console.
     2. Navigate to `Launch Templates` and click `Create Launch Template`.
     3. Specify template details including AMI ID, instance type, security group, and IAM role.

3. **Create Auto Scaling Group**:
   - **Via AWS Console**:
     1. Open the EC2 Management Console.
     2. Navigate to `Auto Scaling Groups` and click `Create Auto Scaling Group`.
     3. Specify the name, launch template, VPC, and target group.
     4. Configure health checks, scaling policies, and notifications.

4. **Validate Auto Scaling**:
   - **Command**:
     ```bash
     aws autoscaling describe-auto-scaling-groups --auto-scaling-group-names <auto-scaling-group-name>
     ```
   - **Explanation**:
     - `aws autoscaling describe-auto-scaling-groups`: Describes the auto-scaling group settings.

## Validation and Summary

1. **Validate Setup**:
   - Access the application URL through the load balancer to ensure that the application is functional.
   - **Command**:
     ```bash
     curl -I https://<load-balancer-url>
     ```
   - **Explanation**:
     - `curl -I`: Fetches HTTP headers to verify the response from the load balancer.

2. **Summary**:
   - The application is accessible through an HTTPS load balancer.
   - The load balancer forwards requests to Tomcat EC2 instances.
   - EC2 instances are automatically scaled based on load.

## Future Improvements

- **CI/CD Integration**: Implement a CI/CD pipeline for automated artifact deployment.
- **Managed Services**: Consider migrating to AWS managed services like RDS for MySQL, ElastiCache for Memcached, and Amazon MQ for RabbitMQ.

 

---
