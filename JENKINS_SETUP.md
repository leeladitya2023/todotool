# Jenkins Pipeline Setup - Step by Step Guide

Complete guide for setting up your CI/CD pipeline in Jenkins.

---

## Part 1: Install Jenkins

### Option A: Install Jenkins on Windows

1. **Download Jenkins**:
   - Go to: https://www.jenkins.io/download/
   - Click "Windows" → Download Jenkins installer
   - Run the installer (jenkins.msi)

2. **Initial Setup**:
   - Open browser: http://localhost:8080
   - Get initial admin password from: `C:\Program Files\Jenkins\secrets\initialAdminPassword`
   - Or check Jenkins logs
   - Enter password and click "Continue"

3. **Install Plugins**:
   - Select "Install suggested plugins"
   - Wait for installation to complete

4. **Create Admin User**:
   - Enter username, password, email
   - Click "Save and Finish"
   - Click "Start using Jenkins"

### Option B: Run Jenkins in Docker (Alternative)

```bash
docker run -d --name jenkins -p 8080:8080 -v jenkins-data:/var/jenkins_home jenkins/jenkins:lts

# Get password (wait 30 seconds first)
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Then follow steps 2-4 above.

---

## Part 2: Install Required Plugins

### Step 1: Open Plugin Manager
1. In Jenkins, click **"Manage Jenkins"** (left sidebar)
2. Click **"Manage Plugins"**

### Step 2: Install Required Plugins

Go to **"Available"** tab and search for these plugins:

#### Essential Plugins:
- ✅ **Pipeline** (usually pre-installed)
- ✅ **Git** (usually pre-installed)
- ✅ **NodeJS Plugin** (for Node.js support)
- ✅ **Docker Pipeline** (if using Docker)
- ✅ **SSH Pipeline Steps** (if deploying to server)

#### Recommended Plugins:
- ✅ **HTML Publisher** (for test reports)
- ✅ **GitHub** (for GitHub integration)
- ✅ **Blue Ocean** (nice UI for pipelines)

**Install Steps:**
1. Check the boxes next to plugins
2. Click **"Install without restart"**
3. After installation, click **"Restart Jenkins when installation is complete"**

---

## Part 3: Configure Node.js (If Needed)

### Step 1: Install Node.js Plugin (if not done)
- Go to: **Manage Jenkins** → **Manage Plugins** → **Available**
- Search: "NodeJS Plugin"
- Install

### Step 2: Configure Node.js
1. Go to: **Manage Jenkins** → **Global Tool Configuration**
2. Scroll to **"NodeJS"**
3. Click **"Add NodeJS"**
4. Fill in:
   - Name: `NodeJS-18`
   - Version: Select latest LTS (18.x or 20.x)
   - Check **"Install automatically"**
5. Click **"Save"**

---

## Part 4: Configure Credentials (Optional - Only if needed)

### Configure Docker Credentials (Only if using Jenkinsfile with Docker registry)

1. Go to: **Manage Jenkins** → **Manage Credentials**
2. Click **"Jenkins"** → **"Global"** → **"Add Credentials"**
3. Fill in:
   - **Kind**: Username with password
   - **Username**: Your Docker Hub username
   - **Password**: Your Docker Hub password
   - **ID**: `docker-credentials` (use this exact ID in Jenkinsfile)
   - **Description**: "Docker Hub Credentials"
4. Click **"Create"**

### Configure SSH Credentials (Only if deploying to a server)

1. **Generate SSH Key** (if you don't have one):
   ```bash
   ssh-keygen -t rsa -b 4096 -C "jenkins@deploy"
   # Save to: C:\Users\91891\.ssh\jenkins_deploy
   # Press Enter for no passphrase
   ```

2. **Copy public key to server**:
   ```bash
   type C:\Users\91891\.ssh\jenkins_deploy.pub | ssh user@server "cat >> ~/.ssh/authorized_keys"
   ```

3. **Add SSH Credentials in Jenkins**:
   - Go to: **Manage Jenkins** → **Manage Credentials**
   - Click **"Jenkins"** → **"Global"** → **"Add Credentials"**
   - Fill in:
     - **Kind**: SSH Username with private key
     - **Username**: Your server username (e.g., `ubuntu`)
     - **Private Key**: Click **"Enter directly"** → Paste content of `C:\Users\91891\.ssh\jenkins_deploy`
     - **ID**: `ssh-credentials` (use this exact ID in Jenkinsfile)
     - **Description**: "SSH Deployment Credentials"
   - Click **"Create"**

**Note**: If you don't have a deployment server, skip this step.

---

## Part 5: Create Pipeline Job

### Step 1: Create New Pipeline
1. In Jenkins dashboard, click **"New Item"** (left sidebar)
2. **Item name**: `todo-reduxtoolkit-pipeline` (or any name you prefer)
3. Select **"Pipeline"**
4. Click **"OK"**

### Step 2: Configure General Settings
1. **Description**: "CI/CD Pipeline for Redux Toolkit Todo App"
2. (Optional) Check **"GitHub project"** and enter your GitHub repo URL:
   - Project url: `https://github.com/YOUR_USERNAME/todo-reduxtoolkit`

### Step 3: Configure Build Triggers (Optional)

#### Option A: Manual Build (Start Here)
- Leave everything unchecked
- You'll manually click "Build Now" to trigger builds

#### Option B: Automatic Build on Push (Recommended)
1. Check **"GitHub hook trigger for GITScm polling"**
2. Or check **"Poll SCM"** with schedule: `H/5 * * * *` (checks every 5 minutes)

#### Option C: Scheduled Build
- Check **"Build periodically"**
- Schedule: `H 2 * * *` (daily at 2 AM)

### Step 4: Configure Pipeline Section (IMPORTANT!)

1. Scroll to **"Pipeline"** section
2. **Definition**: Select **"Pipeline script from SCM"**
3. **SCM**: Select **"Git"**
4. **Repository URL**: Enter your GitHub repository URL
   ```
   https://github.com/YOUR_USERNAME/todo-reduxtoolkit.git
   ```
   Or if your repo is in Ecohub:
   ```
   https://github.com/YOUR_USERNAME/Ecohub.git
   ```

5. **Credentials**: 
   - Leave empty if repository is **Public**
   - Add credentials if repository is **Private**
   - To add: Click "Add" → Kind: "Username with password" → Enter GitHub username/password

6. **Branches to build**: 
   - Branch Specifier: `*/main` (or `*/master` if your main branch is `master`)

7. **Script Path** (CRITICAL!):
   - If your Jenkinsfile is at: `REACT/11todoReduxToolkit/Jenkinsfile.local`
   - Enter: `REACT/11todoReduxToolkit/Jenkinsfile.local`
   
   **OR** if you copied it to root:
   - Enter: `Jenkinsfile.local`

### Step 5: Save Pipeline
1. Click **"Save"** at the bottom

---

## Part 6: Run Your First Build

### Step 1: Trigger Build
1. Click on your pipeline job: `todo-reduxtoolkit-pipeline`
2. Click **"Build Now"** (left sidebar)
3. You'll see a build appear in the **"Build History"** (bottom left)

### Step 2: View Build Progress
1. Click on the build number (#1, #2, etc.)
2. Click **"Console Output"** (left sidebar)
3. Watch the build progress in real-time

### Step 3: Check Build Status
- **Blue ball** = Success ✅
- **Red ball** = Failed ❌
- **Yellow ball** = Unstable ⚠️

---

## Part 7: Verify Pipeline Works

### Check Each Stage:

Your pipeline should execute these stages:

1. ✅ **Checkout** - Code is checked out from GitHub
2. ✅ **Install Dependencies** - npm ci runs
3. ✅ **Lint** - ESLint runs
4. ✅ **Build Application** - npm run build creates dist folder
5. ✅ **Docker Build** (if using Jenkinsfile.local)
6. ✅ **Run Container** (if using Jenkinsfile.local)

### View Build Artifacts:
1. Click on your build
2. Click **"Build Artifacts"** (if available)
3. You should see the `dist` folder contents

---

## Part 8: Configure GitHub Webhook (Optional - Auto Build)

### Step 1: Install GitHub Plugin
- Go to: **Manage Jenkins** → **Manage Plugins**
- Install: **"GitHub plugin"** (if not already installed)

### Step 2: Get Jenkins URL
1. Go to: **Manage Jenkins** → **Configure System**
2. Scroll to **"GitHub"**
3. Check **"Specify another hook URL"** (if needed)
4. Note your Jenkins URL: `http://YOUR_IP:8080`

**For local testing, use ngrok:**
```bash
# Install ngrok from https://ngrok.com
ngrok http 8080
# Copy the https URL (e.g., https://abc123.ngrok.io)
```

### Step 3: Configure GitHub Webhook
1. Go to your GitHub repository
2. Click **"Settings"** → **"Webhooks"** → **"Add webhook"**
3. Fill in:
   - **Payload URL**: `http://YOUR_JENKINS_IP:8080/github-webhook/`
     - Or if using ngrok: `https://abc123.ngrok.io/github-webhook/`
   - **Content type**: `application/json`
   - **Which events**: Select **"Just the push event"**
4. Click **"Add webhook"**

### Step 4: Test Webhook
1. Make a small change in your code
2. Commit and push:
   ```bash
   git add .
   git commit -m "Test webhook"
   git push
   ```
3. Check Jenkins - build should start automatically! 🎉

---

## Troubleshooting Common Issues

### Issue 1: "Cannot find Jenkinsfile"
**Solution:**
- Check the **Script Path** in pipeline configuration
- Make sure path matches your file location
- If file is at `REACT/11todoReduxToolkit/Jenkinsfile.local`
- Script Path should be: `REACT/11todoReduxToolkit/Jenkinsfile.local`

### Issue 2: "npm: command not found"
**Solution:**
- Install NodeJS Plugin (see Part 3)
- Make sure Node.js is installed on Jenkins server
- Or add this to your Jenkinsfile (before npm commands):
  ```groovy
  sh 'which node || echo "Node.js not found"'
  ```

### Issue 3: "Permission denied" or "Access denied"
**Solution:**
- Check file permissions
- If using Docker, make sure Jenkins user can run Docker
- Add Jenkins user to docker group (Linux):
  ```bash
  sudo usermod -aG docker jenkins
  sudo systemctl restart jenkins
  ```

### Issue 4: "Git repository not found"
**Solution:**
- Check repository URL is correct
- If private repo, add GitHub credentials in Jenkins
- Test URL in browser to make sure it's accessible

### Issue 5: "Path not found" or "cd: no such file or directory"
**Solution:**
- Check where Jenkins checks out your code
- If Jenkins checks out to repo root, use: `cd REACT/11todoReduxToolkit`
- If Jenkins checks out to project folder, remove `cd` command
- Check console output to see current directory:
  ```groovy
  sh 'pwd'  // Add this to see current directory
  ```

### Issue 6: Build fails at "npm ci"
**Solution:**
- Check if `package-lock.json` exists in repository
- If not, commit it:
  ```bash
  cd REACT/11todoReduxToolkit
  npm install  # This creates package-lock.json
  git add package-lock.json
  git commit -m "Add package-lock.json"
  git push
  ```

---

## Quick Reference Commands

### Start/Stop Jenkins (Windows):
```bash
# Start Jenkins service
net start Jenkins

# Stop Jenkins service
net stop Jenkins
```

### Access Jenkins:
- URL: http://localhost:8080
- Default port: 8080

### Jenkins Directory (Windows):
- Installation: `C:\Program Files\Jenkins`
- Workspace: `C:\Program Files\Jenkins\workspace`
- Logs: `C:\Program Files\Jenkins\logs`

### View Jenkins Logs:
1. Go to: **Manage Jenkins** → **System Log**
2. Or check: `C:\Program Files\Jenkins\logs\jenkins.log`

---

## Success Checklist

After setup, you should have:

- [x] Jenkins installed and running
- [x] Required plugins installed
- [x] Pipeline job created
- [x] Pipeline job configured with correct Script Path
- [x] First build completed successfully
- [x] Build artifacts archived (dist folder)
- [x] (Optional) GitHub webhook configured for auto-build

---

## Next Steps After Setup

1. ✅ Test your pipeline by making code changes
2. ✅ Check build artifacts after each build
3. ✅ Monitor build history and trends
4. ✅ Set up email notifications (optional)
5. ✅ Configure multiple environments (dev, staging, prod)

---

**Congratulations! Your Jenkins pipeline is now set up! 🎉**

**Need help? Check the console output - it shows exactly where the error occurs!**

