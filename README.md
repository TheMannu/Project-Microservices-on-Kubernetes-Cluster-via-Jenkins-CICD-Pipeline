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

2. **Create an EKS Cluster:**
   - Run the following command to create an EKS cluster:
     ```bash
     eksctl create cluster --name=EKS-1 --region=ap-south-1 --zones=ap-south-1a,ap-south-1b --without-nodegroup
     ```

3. **Associate OIDC Provider:** → An OIDC provider (OpenID Connect provider) is like an identity verification service for applications. It ensures that people or systems accessing our application or cloud resources are who they claim to be.

- Our ID card is the identity token.
- The OIDC provider is the trusted system (the database) that confirms our ID is valid.

   - Associate an OIDC provider with the cluster:
     ```bash
     eksctl utils associate-iam-oidc-provider --region ap-south-1 --cluster EKS-1 --approve
     ```

4. **Create a Node Group:**
   - Create a node group for the EKS cluster:
     ```bash
     eksctl create nodegroup --cluster=EKS-1 --region=ap-south-1 --name=node2 --node-type=t3.medium --nodes=3 --nodes-min=2 --nodes-max=4 --node-volume-size=20 --ssh-access --ssh-public-key=DevOps --managed --asg-access --external-dns-access --full-ecr-access --appmesh-access --alb-ingress-access
     ```

5. **Update Kubeconfig:** →This command is used to configure your local Kubernetes kubectl command-line tool to interact with an Amazon EKS (Elastic Kubernetes Service) cluster.

   - Update the kubeconfig file to interact with the EKS cluster:
     ```bash
     aws eks update-kubeconfig --region ap-south-1 --name EKS-1
     ```

6. **Create a Namespace:** A namespace in Kubernetes is like a folder for organizing resources. It helps you group and manage Kubernetes objects (like pods, services, or secrets) that belong to the same team, project, or environment.

   - Create a namespace for the microservices:
     ```bash
     kubectl create namespace webapps
     kubectl get namespaces
     ```

7. **Create a Service Account `(svc-acc.yaml)`:** A Service Account in Kubernetes is like a special user for applications running in your cluster. It allows pods (your running applications) to interact with the Kubernetes API or other systems securely.

   - Create a service account for Jenkins:

     ```bash
     vim svc-acc.yaml
     ```

     ```yaml
     apiVersion: v1
     kind: ServiceAccount
     metadata:
       name: jenkins
       namespace: webapps
     ```

     Apply the configuration:
     ```bash
     kubectl apply -f svc-acc.yaml
     ```
     
8. **Create a Role `(app-role.yaml)`:** It is used to define a Role or RoleBinding for an application in your cluster. It specifies what actions the application (or a service account used by the application) is allowed to perform on specific resources within a namespace.
A Role in Kubernetes grants permissions within a single namespace.
It specifies what actions (like get, list, create, delete) are allowed on certain resources (like Pods, Services, ConfigMaps).

   - Define a role for the service account:

     ```bash
     vim app-role.yaml
     ```

     ```yaml
     apiVersion: rbac.authorization.k8s.io/v1
     kind: Role
     metadata:
       name: app-role
       namespace: webapps
     rules:
       - apiGroups: ["", "apps", "autoscaling", "batch", "extensions", "policy", "rbac.authorization.k8s.io"]
         resources: ["pods", "componentstatuses", "configmaps", "daemonsets", "deployments", "events", "endpoints", "horizontalpodautoscalers", "ingress", "jobs", "limitranges", "namespaces", "nodes", "pods", "persistentvolumes", "persistentvolumeclaims", "resourcequotas", "replicasets", "replicationcontrollers", "serviceaccounts", "services"]
         verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
     ```

     Apply the configuration:
     ```bash
     kubectl apply -f app-role.yaml
     ```

9. **Bind the Role to the Service Account `(role-bind.yaml)`:** A RoleBinding binds a Role to a user, group, or service account, allowing them to use the permissions defined in the Role.

   - Create a role binding:
     ```bash
     vim role-bind.yaml
     ```

     ```yaml
     apiVersion: rbac.authorization.k8s.io/v1
     kind: RoleBinding
     metadata:
       name: app-rolebinding
       namespace: webapps
     roleRef:
       apiGroup: rbac.authorization.k8s.io
       kind: Role
       name: app-role
     subjects:
       - kind: ServiceAccount
         name: jenkins
         namespace: webapps
     ```

     Apply the configuration:
     ```bash
     kubectl apply -f role-bind.yaml
     ```

10. **Create a Secret Token by utilizing a service account in the desired namespace `(secret.yaml)`:** A secret token in Kubernetes is used to allow a pod or application to securely prove its identity and access resources it has permission for. It’s like a password or keycard for your application.

    - Create a secret token for the service account:

     ```bash
     vim secret.yaml
     ```

      ```yaml
      apiVersion: v1
      kind: Secret
      type: kubernetes.io/service-account-token
      metadata:
        name: mysecretname
        annotations:
          kubernetes.io/service-account.name: jenkins
      ```

      Apply the configuration:
      ```bash
      kubectl apply -f secret.yaml -n webapps
      ```

    - Retrieve the token:
      ```bash
      kubectl describe secret mysecretname -n webapps
      ```

    - Copy the token and save it for later use in Jenkins Configuration.

---

### **Step 4: Install Essential Tools**
1. **Run the Following Commands:**
   - Update the system:
     ```bash
     sudo su
     apt update
     sudo apt-get install curl -y
     sudo apt-get install unzip -y
     ```

2. **Create a Shell Script to Install Tools:**
   - Create a file `install.sh`:
     ```bash
     #!/bin/bash
     sudo apt update -y
     wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
     echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
     sudo apt update -y
     sudo apt install temurin-17-jdk -y
     /usr/bin/java --version

     # Install Jenkins
     curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
     echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
     sudo apt-get update -y
     sudo apt-get install jenkins -y
     sudo systemctl start jenkins
     sudo systemctl status jenkins
     sudo systemctl enable jenkins

     # Install Docker
     sudo apt-get update
     sudo apt-get install docker.io -y
     sudo usermod -aG docker ubuntu
     sudo usermod -aG docker jenkins
     newgrp docker
     sudo chmod 777 /var/run/docker.sock
     sudo systemctl restart jenkins

     # Install AWS CLI
     curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
     sudo apt-get install unzip -y
     unzip awscliv2.zip
     sudo ./aws/install

     # Install kubectl
     sudo apt update
     sudo apt install curl -y
     curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
     sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
     kubectl version --client

     # Install eksctl
     curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
     sudo mv /tmp/eksctl /usr/local/bin
     eksctl version
     ```
3. **Execute the Script:**
   - Run the script:
     ```bash
     chmod +x install.sh && ./install.sh
     ```
     ```
     sudo chmod +x install.sh
     sudo bash install.sh
     ```
---

### **Step 5: Set Up Jenkins**
1. **Access Jenkins:**
   - Open your browser and go to `http://<public-ip>:8080` (to access jenkins console).
   - Retrieve the initial admin password:
     ```bash
     sudo cat /var/lib/jenkins/secrets/initialAdminPassword
     ```
   - Paste the password into the Jenkins setup page.

2. **Install Suggested Plugins:**
   - Install the recommended plugins during the Jenkins setup.

3. **Create a Jenkins Admin User:**
   - Set up a username and password for Jenkins.

4. **Install Additional Plugins:**
   - Install the following plugins:
     - Docker (all plugins)
     - Kubernetes (all plugins)
     - Multibranch Scan Webhook Trigger

5. **Set Up Credentials:**
   - **Docker Hub Credentials:**
     - Go to `Manage Jenkins > Credentials > Global > Add Credentials`.
     - Provide your Docker Hub username and password. Set the ID as `id==docker`.

   - **GitHub Credentials:**
     - Generate a GitHub token and add it to Jenkins with the ID `id==github`.

   - **Kubernetes Token:**
     - Add the Kubernetes token (saved earlier) as a secret text with the ID `id==k8-token`.

---

### **Step 6: Create Jenkins Pipeline**
1. **Create a Multibranch Pipeline:**
   - Go to Jenkins and click `New Item`.
   - Select `Multibranch Pipeline`.

2. **Configure the Pipeline:**
   - Choose git from Add Source option and paste the Project `Repository URL` there
   - Add the GitHub repository URL.
   - Select the GitHub credentials from the dropdown.

3. **Set Up a Webhook in github to trigger the pipeline:**
   - Go to your GitHub repository settings -> webhook -> Add a webhook.