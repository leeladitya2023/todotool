# Jenkinsfile Configuration Guide

This document explains what details you need to fill in for each Jenkinsfile.

---

## 📝 Jenkinsfile.local (Simplified - No Deployment Server Required)

### ✅ Already Configured - No Changes Needed!

The `Jenkinsfile.local` is ready to use as-is. It works for:
- Local Jenkins setup
- No deployment server needed
- Uses Docker for building and running containers locally

**All variables are already set:**
- `DOCKER_IMAGE = 'todo-reduxtoolkit'` ✅ (You can change if you want)
- `DOCKER_TAG = "${env.BUILD_NUMBER}"` ✅ (Auto-generated)
- `CONTAINER_NAME = 'todo-reduxtoolkit-app'` ✅ (You can change if you want)

**The path assumes your repo structure:**
- If Jenkins job points to project root: Paths include `REACT/11todoReduxToolkit/` ✅
- If Jenkins job points to `REACT/11todoReduxToolkit/`: Remove `cd REACT/11todoReduxToolkit` lines

---

## 📝 Jenkinsfile (Full Version - With Deployment)

### Required Configuration (Must Fill In):

#### 1. Docker Registry Settings (Lines 8-9)
```groovy
DOCKER_REGISTRY = 'your-registry.com' // ⚠️ CHANGE THIS
DOCKER_CREDENTIALS = 'docker-credentials' // ⚠️ CHANGE THIS
```

**What to fill:**

**Option A: Using Docker Hub**
```groovy
DOCKER_REGISTRY = 'docker.io'  // or just 'docker.io'
DOCKER_CREDENTIALS = 'docker-hub-creds'  // Jenkins credential ID
```

**Option B: Using Private Registry**
```groovy
DOCKER_REGISTRY = 'registry.example.com'
DOCKER_CREDENTIALS = 'my-docker-creds'  // Jenkins credential ID
```

**How to create Docker credentials in Jenkins:**
1. Go to: **Manage Jenkins** → **Manage Credentials**
2. Click **Jenkins** → **Global** → **Add Credentials**
3. Kind: **Username with password**
4. Username: Your Docker Hub/Registry username
5. Password: Your Docker Hub/Registry password
6. ID: `docker-hub-creds` (use this same ID in Jenkinsfile)
7. Click **Create**

---

#### 2. Deployment Server Settings (Lines 16-18)
```groovy
DEPLOY_HOST = 'your-deployment-host.com' // ⚠️ CHANGE THIS
DEPLOY_USER = 'deploy' // ⚠️ CHANGE THIS
DEPLOY_SSH_CREDENTIALS = 'ssh-credentials' // ⚠️ CHANGE THIS
```

**What to fill:**

**Example for AWS EC2:**
```groovy
DEPLOY_HOST = 'ec2-12-34-56-78.compute-1.amazonaws.com'  // Your server IP/hostname
DEPLOY_USER = 'ubuntu'  // or 'ec2-user' for Amazon Linux
DEPLOY_SSH_CREDENTIALS = 'aws-ssh-key'  // Jenkins credential ID
```

**Example for DigitalOcean:**
```groovy
DEPLOY_HOST = '157.230.123.45'  // Your droplet IP
DEPLOY_USER = 'root'  // or 'deploy'
DEPLOY_SSH_CREDENTIALS = 'do-ssh-key'  // Jenkins credential ID
```

**Example for Local Server:**
```groovy
DEPLOY_HOST = '192.168.1.100'  // Local IP
DEPLOY_USER = 'your-username'
DEPLOY_SSH_CREDENTIALS = 'local-ssh-key'  // Jenkins credential ID
```

**How to create SSH credentials in Jenkins:**
1. **Generate SSH key** (if you don't have one):
   ```bash
   ssh-keygen -t rsa -b 4096 -C "jenkins@deploy"
   # Save to: ~/.ssh/jenkins_deploy
   # No passphrase (or set one)
   ```

2. **Copy public key to server:**
   ```bash
   ssh-copy-id -i ~/.ssh/jenkins_deploy.pub user@your-server.com
   ```

3. **Add SSH credentials in Jenkins:**
   - Go to: **Manage Jenkins** → **Manage Credentials**
   - Click **Jenkins** → **Global** → **Add Credentials**
   - Kind: **SSH Username with private key**
   - Username: `ubuntu` (or your server user)
   - Private Key: **Enter directly** → Paste content of `~/.ssh/jenkins_deploy`
   - ID: `aws-ssh-key` (use this same ID in Jenkinsfile)
   - Click **Create**

---

#### 3. Branch Names (Optional - Lines 136, 156)
```groovy
branch 'develop'  // Line 136 - Change if your staging branch is different
branch 'main'     // Line 156 - Change if your main branch is 'master'
```

**If your branches are different:**
```groovy
branch 'staging'  // Instead of 'develop'
branch 'master'   // Instead of 'main'
```

---

#### 4. Application Port (Line 12)
```groovy
APP_PORT = '3000'  // Usually fine as-is, change if you need different port
```

---

## 🔧 Path Configuration

### Important: Check Your Jenkins Job Configuration!

If your Jenkins job **points to the repository root** (Ecohub/):
- ✅ Keep paths as-is in `Jenkinsfile.local` (includes `REACT/11todoReduxToolkit/`)

If your Jenkins job **points to REACT/11todoReduxToolkit/**:
- ❌ Remove `cd REACT/11todoReduxToolkit` from `Jenkinsfile.local`
- Update all paths accordingly

**To check/fix:**

**In Jenkinsfile.local** (lines 28, 40, 52, 69):
Currently:
```groovy
cd REACT/11todoReduxToolkit
npm ci
```

If Jenkins workspace is already in the project folder, change to:
```groovy
npm ci  // Remove the cd line
```

**In Jenkinsfile** (main file):
Currently uses root paths (assumes workspace is in project folder):
```groovy
npm ci  // Already correct if workspace is in project folder
```

---

## ✅ Quick Configuration Checklist

### For Jenkinsfile.local (Simple Setup):
- [ ] Check if Jenkins workspace path needs adjustment (see Path Configuration above)
- [ ] Optional: Change `DOCKER_IMAGE` name if you want
- [ ] Optional: Change `CONTAINER_NAME` if you want
- [x] **DONE!** Ready to use

### For Jenkinsfile (Full Setup):
- [ ] Fill in `DOCKER_REGISTRY` (line 8)
- [ ] Create Docker credentials in Jenkins
- [ ] Fill in `DOCKER_CREDENTIALS` ID (line 9)
- [ ] Fill in `DEPLOY_HOST` (line 16)
- [ ] Fill in `DEPLOY_USER` (line 17)
- [ ] Create SSH credentials in Jenkins
- [ ] Fill in `DEPLOY_SSH_CREDENTIALS` ID (line 18)
- [ ] Check branch names match your repo (lines 136, 156)
- [ ] Check Jenkins workspace path (see Path Configuration above)

---

## 📋 Example Complete Configuration

### Minimal Jenkinsfile.local (Already Ready):
```groovy
environment {
    DOCKER_IMAGE = 'todo-reduxtoolkit'  // ✅ OK
    DOCKER_TAG = "${env.BUILD_NUMBER}"   // ✅ Auto-generated
    CONTAINER_NAME = 'todo-reduxtoolkit-app'  // ✅ OK
}
```

### Complete Jenkinsfile Configuration Example:
```groovy
environment {
    DOCKER_IMAGE = 'todo-reduxtoolkit'
    DOCKER_TAG = "${env.BUILD_NUMBER}"
    DOCKER_REGISTRY = 'docker.io'  // ✅ Filled
    DOCKER_CREDENTIALS = 'my-docker-creds'  // ✅ Filled
    
    APP_PORT = '3000'
    CONTAINER_NAME = 'todo-reduxtoolkit-app'
    
    DEPLOY_HOST = '52.91.123.45'  // ✅ Filled (your server IP)
    DEPLOY_USER = 'ubuntu'  // ✅ Filled
    DEPLOY_SSH_CREDENTIALS = 'my-ssh-key'  // ✅ Filled
}
```

---

## 🚫 Don't Have Deployment Server?

**Use Jenkinsfile.local instead!** It doesn't require:
- ❌ Docker registry (builds locally)
- ❌ Deployment server
- ❌ SSH credentials

Just works out of the box for local CI/CD testing.

---

## 🔍 How to Test Configuration

1. **Save your Jenkinsfile**
2. **Commit and push to GitHub:**
   ```bash
   git add Jenkinsfile
   git commit -m "Configure Jenkins pipeline"
   git push
   ```
3. **In Jenkins, click "Build Now"**
4. **Check console output for errors**
5. **If errors about credentials → Check credential IDs match**
6. **If errors about paths → Check workspace configuration**

---

**Need help? Check the console output in Jenkins - it will tell you exactly what's missing!**

