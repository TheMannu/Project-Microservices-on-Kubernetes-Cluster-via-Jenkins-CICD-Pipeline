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

2. **Connect to the EC2 Instance via SSH:**