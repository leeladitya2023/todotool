# Jenkins Pipeline Quick Setup (5 Minutes)

## 🚀 Quick Steps

### 1. Install Jenkins
- Download: https://www.jenkins.io/download/
- Run installer (Windows)
- Access: http://localhost:8080
- Enter password from installer/console

### 2. Install Plugins
- Go to: **Manage Jenkins** → **Manage Plugins** → **Available**
- Install: **Pipeline**, **Git**, **Docker Pipeline**
- Restart Jenkins

### 3. Create Pipeline Job
1. Click **"New Item"**
2. Name: `todo-pipeline`
3. Select **"Pipeline"**
4. Click **"OK"**

### 4. Configure Pipeline
1. Scroll to **"Pipeline"** section
2. **Definition**: Pipeline script from SCM
3. **SCM**: Git
4. **Repository URL**: `https://github.com/YOUR_USERNAME/YOUR_REPO.git`
5. **Branch**: `*/main`
6. **Script Path**: `REACT/11todoReduxToolkit/Jenkinsfile.local`

### 5. Save & Build
1. Click **"Save"**
2. Click **"Build Now"**
3. Check console output

---

## ✅ That's It!

Your pipeline will:
- ✅ Checkout code
- ✅ Install dependencies
- ✅ Run linting
- ✅ Build app
- ✅ Build Docker image (if using Jenkinsfile.local)

---

## 🔧 Quick Fixes

**Problem: Can't find Jenkinsfile**
→ Check Script Path matches your file location

**Problem: npm not found**
→ Install NodeJS Plugin in Jenkins

**Problem: Build fails**
→ Check console output for error details

---

**For detailed guide, see: JENKINS_SETUP.md**

