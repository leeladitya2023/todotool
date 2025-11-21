# Complete CI/CD Setup Guide - Step by Step

This guide will walk you through setting up your Redux Toolkit project on GitHub and configuring Jenkins CI/CD with Docker.

---

## Part 1: Push Project to GitHub

### Step 1: Initialize Git Repository (if not already done)
```bash
cd REACT/11todoReduxToolkit
git init
```

### Step 2: Check Current Git Status
```bash
git status
```

### Step 3: Stage All Files
```bash
git add .
```

### Step 4: Create Initial Commit
```bash
git commit -m "Initial commit: Redux Toolkit Todo app with CI/CD pipeline"
```

### Step 5: Create GitHub Repository
1. Go to https://github.com
2. Click the "+" icon in the top right
3. Select "New repository"
4. Repository name: `todo-reduxtoolkit` (or any name you prefer)
5. Description: "Redux Toolkit Todo Application with Jenkins CI/CD"
6. Choose Public or Private
7. **DO NOT** initialize with README, .gitignore, or license (you already have files)
8. Click "Create repository"

### Step 6: Connect Local Repository to GitHub
```bash
# Add remote (replace YOUR_USERNAME with your GitHub username)
git remote add origin https://github.com/YOUR_USERNAME/todo-reduxtoolkit.git

# Rename branch to main (if needed)
git branch -M main

# Push to GitHub
git push -u origin main
```

---

## Part 2: Install and Setup Docker

### Step 7: Install Docker Desktop
1. Download Docker Desktop for Windows: https://www.docker.com/products/docker-desktop
2. Install and restart your computer
3. Start Docker Desktop
4. Verify installation:
```bash
docker --version
docker-compose --version
```

### Step 8: Test Docker Build Locally
```bash
cd REACT/11todoReduxToolkit

# Build the Docker image
docker build -t todo-reduxtoolkit:latest .

# Run the container
docker run -d -p 3000:80 --name todo-app todo-reduxtoolkit:latest

# Check if it's running
docker ps

# Visit http://localhost:3000 in your browser
# To stop: docker stop todo-app
# To remove: docker rm todo-app
```

---

## Part 3: Install and Setup Jenkins

### Step 9: Install Jenkins
#### Option A: Using Docker (Recommended)
```bash
# Create Jenkins data directory
mkdir jenkins-data

# Run Jenkins in Docker
docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins-data:/var/jenkins_home \
  jenkins/jenkins:lts

# Get initial admin password
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

#### Option B: Install Jenkins on Windows
1. Download Jenkins installer: https://www.jenkins.io/download/
2. Choose "Windows" and download the installer
3. Run the installer and follow setup wizard
4. Access Jenkins at http://localhost:8080

### Step 10: Initial Jenkins Setup
1. Open browser: http://localhost:8080
2. Enter the initial admin password (from Step 9)
3. Click "Install suggested plugins"
4. Wait for installation
5. Create admin user (username, password, email)
6. Click "Save and Finish"

### Step 11: Install Required Jenkins Plugins
1. Go to: **Manage Jenkins** → **Manage Plugins** → **Available**
2. Search and install these plugins:
   - **Docker Pipeline**
   - **SSH Pipeline Steps**
   - **Git** (usually pre-installed)
   - **GitHub** (for GitHub integration)
   - **Pipeline** (usually pre-installed)
   - **HTML Publisher** (for test reports)
3. Click "Install without restart"
4. After installation, click "Restart Jenkins when installation is complete"

---

## Part 4: Configure Jenkins Credentials

### Step 12: Configure Docker Hub Credentials (if using Docker Hub)
1. Go to: **Manage Jenkins** → **Manage Credentials**
2. Click "Jenkins" → "Global" → "Add Credentials"
3. Kind: **Username with password**
4. Username: Your Docker Hub username
5. Password: Your Docker Hub password
6. ID: `docker-credentials` (or remember what you set)
7. Description: "Docker Hub Credentials"
8. Click "Create"

### Step 13: Configure SSH Credentials (for deployment)
If you have a remote server for deployment:

1. **Generate SSH Key** (if you don't have one):
```bash
ssh-keygen -t rsa -b 4096 -C "jenkins@deploy"
# Save to: C:\Users\91891\.ssh\jenkins_deploy
# Press Enter for no passphrase (or set one)
```

2. **Add SSH Credentials in Jenkins**:
   - Go to: **Manage Jenkins** → **Manage Credentials**
   - Click "Jenkins" → "Global" → "Add Credentials"
   - Kind: **SSH Username with private key**
   - Username: Your server username (e.g., `ubuntu`, `deploy`)
   - Private Key: **Enter directly** → Paste your private key
   - ID: `ssh-credentials` (or remember what you set)
   - Description: "SSH Deployment Credentials"
   - Click "Create"

**Note**: If you don't have a deployment server yet, you can skip this and test locally.

---

## Part 5: Update Jenkinsfile Configuration

### Step 14: Update Jenkinsfile Environment Variables

Open `REACT/11todoReduxToolkit/Jenkinsfile` and update these values:

```groovy
environment {
    DOCKER_IMAGE = 'todo-reduxtoolkit'
    DOCKER_TAG = "${env.BUILD_NUMBER}"
    
    // Update this with your Docker Hub username
    DOCKER_REGISTRY = 'docker.io/YOUR_DOCKERHUB_USERNAME'
    // Or use your own registry: 'your-registry.com'
    
    // Use the credential ID from Step 12
    DOCKER_CREDENTIALS = 'docker-credentials'
    
    APP_PORT = '3000'
    CONTAINER_NAME = 'todo-reduxtoolkit-app'
    
    // Only if you have a deployment server
    DEPLOY_HOST = 'your-server-ip-or-domain.com'
    DEPLOY_USER = 'ubuntu'  // or your server user
    DEPLOY_SSH_CREDENTIALS = 'ssh-credentials'
}
```

**Important**: If you don't have a deployment server, comment out the deployment stages or modify them for local deployment.

### Step 15: Commit and Push Updated Files
```bash
cd REACT/11todoReduxToolkit

git add Jenkinsfile Dockerfile .dockerignore docker-compose.yml nginx.conf
git commit -m "Add CI/CD pipeline configuration"
git push origin main
```

---

## Part 6: Create Jenkins Pipeline Job

### Step 16: Create New Pipeline Job
1. In Jenkins, click **"New Item"**
2. Item name: `todo-reduxtoolkit-pipeline`
3. Select **"Pipeline"**
4. Click **"OK"**

### Step 17: Configure Pipeline
1. **General Settings**:
   - Description: "CI/CD Pipeline for Redux Toolkit Todo App"
   - Check "GitHub project" if you want to link to GitHub
   - Project url: `https://github.com/YOUR_USERNAME/todo-reduxtoolkit`

2. **Build Triggers** (optional):
   - ☑ **"GitHub hook trigger for GITScm polling"** (for auto-build on push)
   - ☑ **"Poll SCM"** with schedule: `H/5 * * * *` (check every 5 minutes)

3. **Pipeline Section**:
   - Definition: **Pipeline script from SCM**
   - SCM: **Git**
   - Repository URL: `https://github.com/YOUR_USERNAME/todo-reduxtoolkit.git`
   - Credentials: Add if repository is private
   - Branch Specifier: `*/main` (or `*/master`)
   - Script Path: `REACT/11todoReduxToolkit/Jenkinsfile`

4. Click **"Save"**

### Step 18: Run Pipeline
1. Click on your pipeline job
2. Click **"Build Now"**
3. Click on the build number in the left sidebar
4. Click **"Console Output"** to see the build progress

---

## Part 7: Test the Pipeline

### Step 19: Verify Build Steps
The pipeline should execute these steps:
1. ✅ Checkout code
2. ✅ Run linting
3. ✅ Run tests
4. ✅ Build application
5. ✅ Build Docker image
6. ✅ Push to Docker registry (if configured)
7. ✅ Security scan

### Step 20: Verify Docker Image
```bash
# Check if image was created
docker images | grep todo-reduxtoolkit

# Run the built image
docker run -d -p 3000:80 --name todo-test todo-reduxtoolkit:BUILD_NUMBER
```

---

## Part 8: Set Up GitHub Webhook (Optional)

### Step 21: Configure GitHub Webhook for Auto-Build
1. Go to your GitHub repository
2. Click **Settings** → **Webhooks** → **Add webhook**
3. Payload URL: `http://YOUR_JENKINS_IP:8080/github-webhook/`
   - If Jenkins is on your local machine, use: `http://your-public-ip:8080/github-webhook/`
   - Or use a service like ngrok: `ngrok http 8080`
4. Content type: `application/json`
5. Which events: **Just the push event**
6. Click **"Add webhook"**

Now every push to GitHub will automatically trigger a Jenkins build!

---

## Troubleshooting

### Issue: Docker build fails
```bash
# Check Docker is running
docker ps

# Check Dockerfile syntax
docker build -t test . --no-cache
```

### Issue: Jenkins can't access Git
- Make sure Git plugin is installed
- Check repository URL is correct
- If private repo, add GitHub credentials in Jenkins

### Issue: Jenkins can't push to Docker Hub
- Verify Docker credentials are correct in Jenkins
- Test login manually: `docker login`

### Issue: Pipeline fails at deployment stage
- If you don't have a deployment server, comment out deployment stages in Jenkinsfile
- Or update to deploy to local Docker

---

## Quick Reference Commands

```bash
# Git commands
git status
git add .
git commit -m "message"
git push origin main

# Docker commands
docker build -t todo-reduxtoolkit .
docker run -d -p 3000:80 todo-reduxtoolkit
docker ps
docker logs todo-app
docker stop todo-app

# Docker Compose
docker-compose up --build
docker-compose down

# Jenkins (if in Docker)
docker exec -it jenkins bash
docker logs jenkins
```

---

## Next Steps

1. ✅ Add more test cases to improve coverage
2. ✅ Configure email notifications in Jenkinsfile
3. ✅ Set up staging and production environments
4. ✅ Add database migrations if needed
5. ✅ Monitor builds with Jenkins Blue Ocean plugin

---

**Congratulations! Your CI/CD pipeline is now set up! 🎉**

