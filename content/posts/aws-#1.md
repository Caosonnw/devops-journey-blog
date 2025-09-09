---
title: "AWS For Beginners #1 | Deploy a Website on AWS EC2 with Docker and NGINX"
date: 2025-01-30T20:00:00+07:00
series: ["AWS"]
categories: ["AWS"]
tags: ["aws", "ec2", "docker", "nginx"]
author: "Cao Sơn"
draft: false
showToc: true
tocOpen: false
cover:
  image: "aws-1/aws-cloud.png"
  alt: "AWS Cloud Infrastructure"
  caption: "Learn to deploy websites on AWS EC2"
  relative: true
  hiddenInList: true
  hiddenInSingle: false
---

# AWS For Beginners #1: Deploy a Website on AWS EC2 with Docker and NGINX

## 🌟 Overview

This comprehensive guide will take you through the complete process of deploying a website on **AWS EC2** using **Docker containerization** and **NGINX reverse proxy**. You'll master the art of leveraging EC2's cloud computing power, Docker's containerization benefits, and NGINX's performance optimization capabilities.

---

## 📋 Prerequisites

Before diving into this tutorial, make sure you have:

- ✅ An active AWS account with appropriate permissions
- ✅ Basic understanding of Linux commands
- ✅ A website ready for deployment
- ✅ Familiarity with Docker concepts

### 📚 Reference Materials

- [Amazon EC2 Fundamentals](https://000004.awsstudygroup.com/en/)

---

## 🚀 Step 1: Create EC2 Ubuntu Instance

### Launch Your EC2 Instance

1. Navigate to the [EC2 Console](https://console.aws.amazon.com/ec2/v2/home)
2. Click **"Launch Instance"**
3. Select **Ubuntu Server** as your operating system

![EC2 Launch Interface](/aws-1/1.png)

### Configure Instance Details

Choose your preferred Ubuntu AMI and instance type based on your project requirements:

![Instance Configuration Screen](/aws-1/2.png)

---

## 🔐 Step 2: Set Up Key Pair Authentication

### Understanding Key Pairs

Key pairs provide secure SSH access to your EC2 instance:

- **🔑 Public Key**: Automatically stored on the EC2 instance
- **🔒 Private Key**: Downloaded to your local machine for authentication

### Create Your Key Pair

1. Click **"Create new key pair"**

![Key Pair Creation Interface](/aws-1/3.png)

2. Configure your key pair settings:
   - Enter a descriptive name for easy identification
   - Select appropriate file format (`.pem` for Linux/Mac, `.ppk` for Windows)
   - Click **"Create key pair"**

![Key Pair Configuration](/aws-1/4.png)

> ⚠️ **Important**: Store your private key securely - it's essential for FileZilla and SSH connections.

---

## 💾 Step 3: Configure Storage

Optimize your EC2 storage configuration for best performance:

- 📏 Adjust storage size within free tier limits
- 🔧 Select appropriate volume type (GP2 recommended for beginners)
- 🔒 Configure encryption if handling sensitive data

![Storage Configuration Panel](/aws-1/5.png)

Complete the launch process by clicking **"Launch Instance"**.

---

## 🖥️ Step 4: Connect to Your Instance

### Access Your EC2 Instance

1. Select your running instance and click **"Connect"**

![Connect to Instance Button](/aws-1/6.png)

2. Choose **EC2 Instance Connect** for convenient web-based terminal access

![Instance Dashboard](/aws-1/7.png)

![Web-based Terminal](/aws-1/8.png)

### Install Docker

Execute the following command to install Docker on your Ubuntu instance:

```bash
sudo snap install docker
```

![Docker Installation Process](/aws-1/9.png)

---

## 📁 Step 5: Set Up File Transfer with FileZilla

### Configure FileZilla Connection

1. Open FileZilla and access **Site Manager**

![FileZilla Site Manager](/aws-1/10.png)

2. Create a new site with these configuration settings:
   - **Protocol**: `SFTP - SSH File Transfer Protocol`
   - **Host**: Your EC2 Public IPv4 DNS
   - **Username**: `ubuntu`
   - **Key file**: Your downloaded `.ppk` file

![FileZilla Configuration Settings](/aws-1/11.png)

3. Copy the Public IPv4 DNS from your EC2 instance dashboard

![EC2 DNS Information](/aws-1/31.png)

![FileZilla Connection Settings](/aws-1/32.png)

4. Connect and accept the host key verification

![Host Key Verification Dialog](/aws-1/13.png)

Transfer your deployment files to the server using simple drag and drop functionality.

---

## 🐳 Step 6: Deploy with Docker

### Build and Run Your Container

1. **Build your Docker image:**

![Docker Build Process](/aws-1/15.png)

2. **Run your Docker container:**

![Docker Container Running](/aws-1/16.png)

---

## 🛡️ Step 7: Configure Security and NGINX

### Update Security Group Rules

1. Navigate to your **EC2 Security Groups**

![Security Groups Panel](/aws-1/17.png)

2. Add a new inbound rule:
   - **Type**: `Custom TCP`
   - **Port**: Your application port (e.g., 3000, 8080)
   - **Source**: `Anywhere (0.0.0.0/0)` for public access

![Security Rule Configuration](/aws-1/18.png)

👉 Successfully deploy the website running on Docker with port other than 3000. However, you may encounter issues with routing between different website paths. This requires proper NGINX configuration.

![Website Deployment Status](/aws-1/20.png)

### Configure NGINX for Optimal Performance

To ensure proper routing and optimal performance, you need to configure NGINX as a reverse proxy:

1. Access your Docker container and edit the NGINX configuration:

```nginx
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Logging configuration
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;

    # Performance optimizations
    sendfile on;
    keepalive_timeout 65;

    # Server configuration
    server {
        listen 80;

        location / {
            root /usr/share/nginx/html;
            index index.html index.htm;
            try_files $uri $uri/ /index.html;
        }
    }
}
```

2. **Restart the container** to apply configuration changes

![NGINX Configuration Applied](/aws-1/21.png)

### 🎉 Verify Your Deployment

Your website should now be accessible with proper routing and performance optimization:

![Working API Endpoints](/aws-1/22.png)

---

## 🧹 Cleanup Resources

### Proper Resource Management

To avoid unnecessary AWS charges, clean up your resources:

#### 1. Remove Key Pair

- Navigate to **EC2 Console → Key Pairs**
- Select and delete your key pair

![Delete Key Pair Process](/aws-1/33.png)
![Confirm Key Pair Deletion](/aws-1/34.png)

#### 2. Delete Custom Security Groups

- Go to **Security Groups**
- Select and delete custom security groups

![Delete Security Group](/aws-1/35.png)
![Confirm Security Group Deletion](/aws-1/36.png)
![Final Security Group Confirmation](/aws-1/37.png)

#### 3. Terminate EC2 Instance

- Select your instance
- Choose **Instance State → Terminate**

![Terminate Instance Process](/aws-1/38.png)
![Confirm Instance Termination](/aws-1/39.png)
![Final Instance Termination](/aws-1/40.png)

---

## 🎯 Conclusion

**Congratulations!** 🎉 You've successfully deployed a containerized website on AWS EC2 with NGINX reverse proxy. This robust setup provides:

- ⚡ **Scalable infrastructure** for growing applications
- 🔒 **Secure deployment** with proper authentication
- 🚀 **High performance** through NGINX optimization
- 📦 **Containerized deployment** for consistency across environments

### 💡 Next Steps

- Monitor your AWS usage through CloudWatch
- Set up automated backups for your application
- Implement CI/CD pipelines for seamless deployments
- Explore auto-scaling groups for handling traffic spikes

> 💰 **Cost Tip**: Always remember to clean up unused resources to avoid unexpected charges!
