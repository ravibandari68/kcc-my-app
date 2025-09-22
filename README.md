# Docker Build & EC2 Deploy via GitHub Actions

This repository contains a GitHub Actions workflow to automate:

1. Building and pushing a Docker image to **Amazon ECR**
2. Triggering a **Deployment on EC2 instances** using **AWS SSM**

---

## 🛠️ Workflow Overview

- **Trigger**: Automatically runs on:
  - Pushes to the `main` branch
  - Manual execution via GitHub Actions UI (`workflow_dispatch`)
- **Build Stage**:
  - Builds a Docker image using the project code
  - Pushes the image to Amazon ECR
- **Deploy Stage**:
  - Finds running EC2 instances tagged with `myapp-ec2-*`
  - Runs a remote shell script using **AWS SSM** to pull and restart the updated container

---

## 📋 Prerequisites

Before using this workflow, make sure the following are set up:

### 🔐 GitHub Secrets stores securely stored in Github enviroment 
| Secret Name              | Description |
|--------------------------|-------------|
| `AWS_ACCESS_KEY_ID`      | Your AWS IAM access key |
| `AWS_SECRET_ACCESS_KEY`  | Your AWS IAM secret key |

> These IAM credentials must have permissions for:
> - ECR (read/write)
> - EC2 (describe instances)
> - SSM (send command)

### 🏷️ EC2 Tag Requirement

Your EC2 instances must have a tag like:


This is used to filter target instances for deployment.

### 📦 EC2 Instance Requirements

- EC2 instances must be:
  - Running
  - In the same region (`us-east-1`)
  - Connected to AWS SSM (i.e., have SSM agent installed and an IAM role allowing `ssm:SendCommand`)
- Docker must be installed on the EC2 instance
- ECR login and container restart logic is handled in the SSM script

---

## 🚀 How It Works

### 🔧 Build & Push to ECR

1. Checkout the source code
2. Configure AWS credentials
3. Build Docker image:
4. Tag and push to ECR:

### ⚙️ Redeploy on EC2

1. Find EC2 instances with tag `myapp-ec2-*` and state `running`
2. Use **AWS SSM** to send a remote shell command to each instance:
- Stop & remove the old container
- Pull the new image from ECR
- Run the container again

### 🐳 SSM Shell Script
```bash
docker stop myapp || true
docker rm myapp || true
aws ecr get-login-password --region us-east-1 | docker login ...
docker run -d --name myapp -p 80:5000 <your-ecr-repo>:latest
