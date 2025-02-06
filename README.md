# Deploying 10 Microservices on Kubernetes Cluster via Jenkins CI/CD Pipeline

## Project Overview
In this project, we will deploy 10 microservices on a Kubernetes cluster using a Jenkins CI/CD pipeline. The process involves setting up a base server, installing necessary tools, configuring a Kubernetes cluster (EKS) on AWS, and creating a Jenkins pipeline to fetch code from GitHub and deploy it on the EKS cluster.

---
## Project Overview
This project focuses on deploying 10 microservices on a Kubernetes cluster using a Jenkins CI/CD pipeline. The steps include:

1. Setting up a base EC2 instance
2. Creating an IAM user with the necessary permissions
3. Installing essential tools (Docker, Jenkins, Java, eksctl, kubectl, AWS CLI, etc.)
4. Setting up an EKS cluster in AWS
5. Configuring Kubernetes resources and RBAC
6. Running a multibranch Jenkins pipeline to fetch code from GitHub and deploy it on EKS

---

## Step-by-Step Guide

### **Step 1: Set Up a Base Instance**
1. **Launch an EC2 Instance:**
   - Go to the AWS Management Console.
   - Launch an EC2 instance with the following configuration:
     - **AMI:** Ubuntu 24
     - **Instance Type:** t2.large
     - **Storage:** 30GB
     - **Open Security Group Ports:** 22 (SSH), 80 (HTTP), 443 (HTTPS), 8080 (Jenkins)
   - Download the key pair (.pem file) for SSH access.

2. **Connect to the EC2 Instance via SSH:**
   - Click on Connect → SSH Client → Copy the provided SSH command.
   - Open your terminal and navigate to the folder where the .pem file is stored.
   - Use the SSH command to connect to the instance:
     ```bash
     ssh -i <your-key-pair>.pem ubuntu@<public-ip>
     ```

---

### **Step 2: Create an IAM User with Specific Permissions**
1. **Create an IAM User:**
   - Go to the AWS IAM Console → IAM → Users → Create User.
   - Create a new user with the following policies attached:
     - `AmazonEC2FullAccess`
     - `AmazonEKSClusterPolicy`
     - `AmazonEKSWorkerNodePolicy`
     - `AWSCloudFormationFullAccess`
     - `IAMFullAccess`
   - Download the user's access key and secret key.

2. **Create an Inline Policy for EKS:**
   - In the IAM Console, Go to IAM → Policies → Create Inline Policy → JSON.
   - Create an inline policy with the following JSON:

     ```json
     {
       "Version": "2012-10-17",
       "Statement": [
         {
           "Effect": "Allow",
           "Action": "eks:*",
           "Resource": "*"
         }
       ]
     }
     ```

3. **Generate Access Keys:**
   - Create Access keys and Secret keys for the IAM user and download them for use in the AWS CLI.
   - Download and securely store the keys for later use
---


### **Step 3: Set Up Kubernetes Cluster (EKS Cluster)**
1. **Configure AWS CLI:**
   - On your EC2 instance, run:
     ```bash
     aws configure
     ```
   - Provide the access key and secret key downloaded earlier.

