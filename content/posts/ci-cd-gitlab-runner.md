---
title: "CI/CD For Beginners #2 | CI/CD with GitLab Runner: Step-by-Step Guide"
date: 2025-09-08T21:40:00+07:00
series: ["CI-CD"]
categories: ["CI-CD"]
tags: ["ubuntu", "devops", "ci-cd"]
author: "Cao Sơn"
draft: false
showToc: true
tocOpen: false
cover:
  image: "ci-cd-2/gitlab-runner-logo.png"
  alt: "Install GitLab server"
  caption: "Install GitLab server"
  relative: true
  hiddenInList: true
  hiddenInSingle: false
---

> **Target**: In this tutorial, you'll learn how to set up and configure _GitLab Runner_ for CI/CD automation. We'll cover installation, registration with GitLab, creating your first pipeline, and implementing automated testing and deployment workflows. By the end, you'll have a fully functional CI/CD pipeline running on your own infrastructure.

---

# Introduction ✨

## What is GitLab Runner? 🤔

Have you ever wondered how your code automatically gets built, tested, and deployed every time you push changes to your repository? The magic happens through GitLab Runner - the essential component that makes CI/CD automation possible.

GitLab Runner serves as the bridge between your GitLab instance and your infrastructure, acting as:

**🔧 The Executor**: Your dedicated worker that runs CI/CD jobs
**🏗️ The Builder**: Compiles and packages your applications
**🧪 The Tester**: Runs automated tests to ensure code quality  
**🚀 The Deployer**: Handles application deployment to various environments
**📊 The Reporter**: Sends job results and logs back to GitLab

## Why Do You Need GitLab Runner? 💡

Without GitLab Runner, your CI/CD pipelines would just be configuration files with no way to execute. GitLab Runner transforms your pipeline definitions into real actions:

- **Automation**: Eliminates manual build and deployment processes
- **Consistency**: Ensures identical execution environments across all runs
- **Speed**: Parallel job execution reduces overall pipeline duration
- **Reliability**: Isolated environments prevent conflicts between jobs
- **Scalability**: Multiple runners can handle increased workload

GitLab Runner is a lightweight, highly-scalable agent that runs your CI/CD jobs and sends the results back to GitLab. It acts as the execution engine for your GitLab CI/CD pipelines, handling the actual building, testing, and deployment of your code.

Think of GitLab Runner as the worker that performs the heavy lifting in your development workflow. When you push code to GitLab, the Runner automatically picks up your pipeline jobs, executes them in isolated environments, and reports back the results. This automation eliminates manual processes and ensures consistent, reliable deployments.

Key roles of GitLab Runner include:

- **Job Execution**: Runs build, test, and deployment scripts defined in your `.gitlab-ci.yml` file
- **Environment Management**: Creates clean, isolated environments for each job execution
- **Scalability**: Supports multiple concurrent jobs and can be distributed across multiple machines
- **Flexibility**: Works with various executors (Docker, Shell, Kubernetes) to match your infrastructure needs

Whether you're a solo developer or part of a large team, GitLab Runner transforms your development process by automating repetitive tasks and ensuring code quality through consistent testing and deployment practices.

# Prerequisites 📋

Before diving into GitLab Runner setup, ensure you have completed the following requirements:

### 1. GitLab Server Installation ✅

Following our previous tutorial **[CI/CD For Beginners #1]**, you should have:

- A running GitLab instance (self-hosted or GitLab.com account)
- Admin access to configure runners
- Basic understanding of GitLab interface

### 2. Project Setup 📁

For this tutorial, I'll use a **Next.js application** with the following structure:

![Nexjs Structure](/ci-cd-2/1.png)

### 3. Sample Application Files 🛠️

**Dockerfile** for containerizing our Next.js app:

```dockerfile
##### Dockerfile #####
FROM node:18-alpine AS base
RUN apk add --no-cache libc6-compat
WORKDIR /app

FROM base AS deps
COPY package.json yarn.lock* package-lock.json* pnpm-lock.yaml* ./
RUN yarn install

FROM base AS builder
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN yarn run build

FROM base AS runner
WORKDIR /app
ENV NODE_ENV=production \
    PORT=3000 \
    HOSTNAME=0.0.0.0 \
    NEXT_TELEMETRY_DISABLED=1

RUN addgroup -g 1001 -S nodejs \
&& adduser -S nextjs -u 1001

# Copy artifacts to run standalone
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static      ./.next/static

USER nextjs

EXPOSE 3000

# Run Next.js application
CMD ["node", "server.js"]
```

> 💡 **Tip**: If you don't have a Next.js project ready, you can use our free template repository that includes all necessary files for this tutorial.

# Getting Started 🚀

## Step 1: Transfer Project Files to Server 📤

First, we need to transfer our zipped project files to the Ubuntu server. We'll use the `scp` command for this task.

### SCP Command Format 📝

```bash
scp source_path [username]@[server_ip]:destination_path
```

**Example:**

```bash
# Transfer zipped project to server
scp ./nextjs-project.zip ubuntu@192.168.1.100:~/

# With SSH key (if required)
scp -i ~/.ssh/id_rsa ./nextjs-project.zip ubuntu@192.168.1.100:~/
```

![Transfer files using scp](/ci-cd-2/2.png)

After successful transfer, let's verify the files on our server:

![Verify files on server](/ci-cd-2/3.png)

## Step 2: Extract Project Files 📦

After successfully transferring files to the server, we need to install `unzip` to extract our project files.

### Install Unzip Utility 🔧

```bash
# Install unzip package
sudo apt install unzip
```

![Install unzip utility](/ci-cd-2/11.png)

### Extract Project Files 📂

Once unzip is installed successfully, extract the project files:

```bash
# Extract the project zip file
unzip nextjs-project.zip
```

![Extract project files](/ci-cd-2/12.png)

## Step 3: Install Docker 🐳

Next, we'll install Docker to run our containerized application. Let's create a dedicated directory and installation script.

### Create Installation Directory 📁

```bash
# Create tools directory for Docker installation
mkdir -p /tools/docker
cd /tools/docker
```

### Create Docker Installation Script 📝

Create the installation script:

```bash
# Create installation script
vi install-docker.sh
```

**Docker Installation Script Content:**

```bash
#!/bin/bash

sudo apt update

sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update

sudo apt install -y docker-ce

sudo systemctl start docker
sudo systemctl enable docker

sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Verify installation
docker --version
docker-compose --version
```

### Execute Installation Script ⚡

Make the script executable and run it:

```bash
# Grant execution permissions
chmod +x install-docker.sh

# Execute the installation script
sh install-docker.sh
```

### Verify Installation ✅

If the installation is successful, you should see Docker and Docker Compose versions displayed:

![Docker installation success](/ci-cd-2/34.png)

> 💡 **Note**: The installation process may take a few minutes depending on your internet connection and server performance.

## Step 4: Create GitLab Group and Project 🏗️

Now we need to access GitLab through the browser and create a group and project for our CI/CD pipeline.

### Create GitLab Group 👥

Navigate to your GitLab instance and create a new group:

![Create GitLab Group](/ci-cd-2/4.png)

### Create New Project 📋

After successfully creating the group, click on "New project" as highlighted in the image below:

![Create New Project](/ci-cd-2/5.png)

### Configure Project Details 📝

Fill in the project information as shown:

![Project Configuration](/ci-cd-2/6.png)

### Project Setup Instructions 🔧

After creating the project successfully, GitLab provides information to configure your Git identity. Make sure to:

- Select **Global** for Git local setup
- Choose **HTTPS** for Add files option

![GitLab Project Setup](/ci-cd-2/7.png)

> 💡 **Important**: Save these Git configuration commands as you'll need them to connect your local repository to GitLab.

## Step 5: Configure Ubuntu Server for GitLab 🖥️

### Update Hosts File 📝

Return to your Ubuntu server and update the hosts file to include your GitLab domain:

```bash
# Edit the hosts file
vi /etc/hosts
```

Add your GitLab domain entry as shown:

![Update hosts file](/ci-cd-2/10.png)

> 💡 **Important**: Make sure to use the same domain configuration as your GitLab server.

### Create Data Directory 📁

Create a dedicated directory for our project data:

```bash
# Create data directory
mkdir /data
cd /data
```

### Configure Git Identity 🔧

Set up your Git configuration with your user details:

![Git configuration](/ci-cd-2/9.png)

```bash
# Configure Git globally
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

### Set Up Project Directory 🗂️

Create the project folder structure:

```bash
# Create project directory (replace 'hoobank' with your project name)
mkdir hoobank
```

### Copy Project Files 📋

Copy all project files from the home directory to the data directory:

```bash
# Copy project files to data directory
sudo cp -rf /home/caosonnw/hoobank-app/* /data/hoobank/
```

> 📌 **Note**: Replace `/home/caosonnw/hoobank-app/` with your actual project path and `hoobank` with your project name.

### Push to GitLab 🚀

Now let's check the git status and push our project to GitLab:

```bash
# Navigate to project directory
cd /data/hoobank

# Initialize git repository
git init

# Add remote origin
git remote add origin https://your-gitlab-instance/your-group/your-project.git

# Check git status
git status

# Add all files
git add .

# Commit changes
git commit -m "feat(project): initial commit"

# Push to main branch
git push --set-upstream origin main
```

### Verify GitLab Push 🔍

After pushing to GitLab, let's verify that our project files have been successfully uploaded:

![Verify successful push to GitLab](/ci-cd-2/16.png)

You should see all your project files now available in the GitLab repository, confirming that the push operation was successful.

## Step 6: Install GitLab Runner 🏃‍♂️

Now let's proceed with installing GitLab Runner on our Ubuntu server. Follow these steps carefully:

### Update Package Repository 📦

First, update the system package repository:

```bash
# Update package lists
apt-get update
```

### Add GitLab Runner Repository 📋

Add the official GitLab Runner repository to your system:

```bash
# Download and execute GitLab Runner repository script
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | bash
```

### Install GitLab Runner ⚡

Install GitLab Runner using apt-get:

```bash
# Install GitLab Runner package
apt-get install gitlab-runner
```

### Verify Installation ✅

After installation completes, verify that GitLab Runner is installed correctly:

```bash
# Check GitLab Runner version
gitlab-runner --version
```

### Confirm User Creation 👤

GitLab Runner installation automatically creates a dedicated user account. You can verify this by checking the `/etc/passwd` file:

```bash
# View the passwd file to confirm gitlab-runner user
cat /etc/passwd
```

You should see a `gitlab-runner` user entry as shown in the image:

![GitLab Runner user verification](/ci-cd-2/19.png)

> 💡 **Success**: The presence of the `gitlab-runner` user confirms that the installation was completed successfully and the service is ready for configuration.

## Step 7: Register GitLab Runner 📝

Now we need to register our GitLab Runner with the GitLab instance. Return to your GitLab project and navigate to CI/CD settings.

### Access Runner Configuration 🔧

Go to your GitLab project and navigate to **Settings** → **CI/CD** → **Runners**, then expand the runners section and select **Create project runner**:

![Create project runner](/ci-cd-2/20.png)

### Configure Runner Tags 🏷️

Enter tags for your runner (e.g., `ubuntu-server`) to make it easier to identify in pipeline configurations.

### Execute Registration Script ⚡

GitLab will provide a registration script. Copy and paste it into your Ubuntu server terminal:

```bash
# Example registration command (use the one provided by GitLab)
gitlab-runner register --url "https://your-gitlab-instance" --token "your-runner-token"
```

### Registration Process 📋

During the registration process, you'll be prompted for several configurations:

- **GitLab instance URL**: Press Enter to keep the default URL
- **Runner name**: Press Enter to use the default name
- **Executor**: Choose `shell` and press Enter

Continue until you see the "Configuration" confirmation message, indicating successful registration.

> 💡 **Important**: Make sure to select `shell` as the executor type for this tutorial setup.

### Verify Runner Registration ✅

After successful registration, you should see your runner connected in GitLab's Runners section:

![Runner successfully connected](/ci-cd-2/21.png)

### Configure GitLab Runner Settings ⚙️

Now let's configure the GitLab Runner settings for optimal performance:

```bash
# Edit GitLab Runner configuration file
vi /etc/gitlab-runner/config.toml
```

In the configuration file, update the `concurrent` setting to allow multiple simultaneous jobs:

![Configure concurrent jobs](/ci-cd-2/22.png)

> 💡 **Configuration Tip**: Setting `concurrent = 4` allows one GitLab Runner to handle 4 projects simultaneously, improving efficiency for multiple pipeline executions.

### Grant Docker Permissions 🐳

Give GitLab Runner permission to use Docker:

```bash
# Add gitlab-runner user to docker group
usermod -aG docker gitlab-runner
```

### Start GitLab Runner Service 🚀

Launch the GitLab Runner service in the background:

```bash
# Start GitLab Runner service with nohup for background execution
nohup gitlab-runner run --working-directory /home/gitlab-runner/ --config /etc/gitlab-runner/config.toml --service gitlab-runner --user gitlab-runner 2>&1 &
```

![Start GitLab Runner service](/ci-cd-2/24.png)

This command starts GitLab Runner as a background process and creates a `nohup.out` log file for monitoring service output.

### Verify Runner Status 🔍

Confirm that GitLab Runner is running properly:

```bash
# Check running GitLab Runner processes
ps -ef | grep gitlab-runner
```

![Verify runner process](/ci-cd-2/25.png)

> ✅ **Success**: If you see the GitLab Runner processes in the output, your runner is successfully running and ready to execute CI/CD jobs.

## Step 8: Create CI/CD Pipeline 🔄

After installing GitLab Runner and successfully connecting it to GitLab, let's start with CI/CD. I'll do this manually through the GitLab interface for better understanding.

### Create Additional Branches 🌿

Create two additional branches: `develop` and `staging`:

![Create branches](/ci-cd-2/28.png)

### Set Default Branch 🎯

Set the `develop` branch as the default branch:

![Set default branch](/ci-cd-2/29.png)

### Configure Branch Protection 🛡️

Configure branch protection rules as shown:

![Configure branch protection](/ci-cd-2/30.png)

My workflow approach: When developers create feature branches, make code changes, and push to GitLab requesting a merge to the `develop` branch, CI/CD will automatically run a pipeline for build/test/deploy/check logs. For simplicity, I'll focus on building Docker images and containers for deployment.

### Create GitLab CI Configuration 📝

Create a `.gitlab-ci.yml` file in the `develop` branch with the following content:

```yaml
stages:
  - deploy

variables:
  DOCKER_IMAGES: img-hoobank
  DOCKER_CONTAINER: cons-hoobank
  IMAGE_TAG: "$CI_COMMIT_SHORT_SHA"

# ===== DEPLOY =====
deploy:
  stage: deploy
  variables:
    GIT_STRATEGY: clone
  tags: [ubuntu-server]
  only: [develop]
  script:
    - docker rm -f "$DOCKER_CONTAINER" || true
    - docker build -t "$DOCKER_IMAGES:$IMAGE_TAG" -t "$DOCKER_IMAGES:latest" .
    - >
      docker run -d --name "$DOCKER_CONTAINER"
      --restart unless-stopped
      -e NODE_ENV=production -e PORT=3000 -e HOSTNAME=0.0.0.0
      -p 3000:3000
      "$DOCKER_IMAGES:$IMAGE_TAG"
    - |
      KEEP_1="$DOCKER_IMAGES:$IMAGE_TAG"
      KEEP_2="$DOCKER_IMAGES:latest"
      mapfile -t OLD_IMGS < <(
        docker images "$DOCKER_IMAGES" --format '{{.Repository}}:{{.Tag}}' \
        | grep -v '<none>' \
        | grep -v "^${KEEP_1}\$" \
        | grep -v "^${KEEP_2}\$"
      )
      if [ "${#OLD_IMGS[@]}" -gt 0 ]; then
        echo "Removing old images: ${OLD_IMGS[*]}"
        docker rmi "${OLD_IMGS[@]}" || true
      else
        echo "No old images to remove."
      fi
      docker image prune -f || true
```

### Test the Pipeline 🧪

Create a feature branch `feature/home`:

![Create feature branch](/ci-cd-2/37.png)

Make some changes to a file:

![Make changes](/ci-cd-2/38.png)

Commit the changes:

![Commit changes](/ci-cd-2/39.png)

### Create Merge Request 🔀

Create a merge request to the `develop` branch:

![Create merge request](/ci-cd-2/40.png)

After accepting the merge:

![Accept merge](/ci-cd-2/42.png)

You'll see a pipeline automatically start and run the deploy job:

![Pipeline running](/ci-cd-2/43.png)

After completion, the job will show as successful:

![Job completed](/ci-cd-2/44.png)

### Verify Deployment 🔍

Return to the Ubuntu server and check Docker:

![Check Docker](/ci-cd-2/45.png)

Access the server IP with port 3000 to see the deployed application:

![Verify web application](/ci-cd-2/46.png)

> ✅ **Success**: Whenever there's a push requesting a merge to the `develop` branch, the CI/CD pipeline will automatically run and build the latest project. This creates a development environment with a running web application on the server before merging code to main for production deployment.

# Conclusion 🎉

Congratulations! You've successfully set up a complete CI/CD pipeline with GitLab Runner. This automated workflow now handles:

- **Automatic Builds**: Every merge request triggers a new build
- **Container Deployment**: Applications are containerized and deployed automatically
- **Environment Management**: Clean separation between development and production
- **Image Cleanup**: Old Docker images are automatically removed to save space

Your CI/CD pipeline is now ready to handle real-world development workflows, ensuring consistent and reliable deployments every time you push code changes.

## Next Steps 🎯

In the next #, we’ll advance further with:

- Jenkins for CI/CD pipeline deployment
- Monitoring and logging solutions
- Multi-environment deployments
- Notification integrations

Stay tuned for the next upgrade! 🧑🏻‍💻
