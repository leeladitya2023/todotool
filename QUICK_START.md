# Quick Start Guide - CI/CD Setup

## 📋 Pre-Checklist

Before you start, make sure you have:
- [ ] GitHub account
- [ ] Docker Desktop installed and running
- [ ] Git installed
- [ ] Jenkins installed (or Docker to run Jenkins)

---

## 🚀 Step-by-Step (5 Minutes Setup)

### Step 1: Push to GitHub
```bash
cd REACT/11todoReduxToolkit
git init
git add .
git commit -m "Add CI/CD pipeline"
git branch -M main

# Create repo on GitHub first, then:
git remote add origin https://github.com/YOUR_USERNAME/todo-reduxtoolkit.git
git push -u origin main
```

### Step 2: Start Jenkins with Docker
```bash
docker run -d --name jenkins -p 8080:8080 -v jenkins-data:/var/jenkins_home jenkins/jenkins:lts

# Get password (wait 30 seconds first)
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

### Step 3: Setup Jenkins (Browser)
1. Open: http://localhost:8080
2. Enter password from Step 2
3. Install suggested plugins
4. Create admin user
5. Install these plugins manually:
   - Docker Pipeline
   - SSH Pipeline Steps

### Step 4: Configure Jenkins Job
1. New Item → Pipeline
2. Name: `todo-pipeline`
3. Pipeline → Definition: Pipeline script from SCM
4. SCM: Git
5. Repository: `https://github.com/YOUR_USERNAME/todo-reduxtoolkit.git`
6. Branch: `*/main`
7. Script Path: `REACT/11todoReduxToolkit/Jenkinsfile.local`

### Step 5: Update Jenkinsfile Path in Jenkins
- If your repo root is at `REACT/11todoReduxToolkit/`, set Script Path to: `Jenkinsfile.local`
- If Jenkins job is pointing to project root, set Script Path to: `REACT/11todoReduxToolkit/Jenkinsfile.local`

### Step 6: Build Now!
Click "Build Now" and watch it run! 🎉

---

## 🧪 Test Locally First

Before pushing to GitHub, test Docker build:

```bash
cd REACT/11todoReduxToolkit
docker build -t todo-reduxtoolkit .
docker run -d -p 3000:80 --name todo-test todo-reduxtoolkit
# Visit http://localhost:3000
```

---

## 🔧 Quick Fixes

### If Docker build fails:
```bash
docker ps  # Make sure Docker is running
docker build -t test . --no-cache  # Test build
```

### If Jenkins can't find Jenkinsfile:
- Check the Script Path in Jenkins job configuration
- Make sure the file is in the correct location

### If you don't have a deployment server:
- Use `Jenkinsfile.local` instead of `Jenkinsfile`
- It will build and run container locally

---

## 📝 Important Notes

1. **For Local Testing**: Use `Jenkinsfile.local`
2. **For Production**: Update `Jenkinsfile` with your Docker Hub credentials
3. **Docker Hub**: Create account at https://hub.docker.com (optional for local testing)

---

## ✅ Success Indicators

- ✅ Jenkins build shows green checkmark
- ✅ Docker image created: `docker images | grep todo`
- ✅ Container running: `docker ps | grep todo`
- ✅ App accessible: http://localhost:3000

---

**Need help? Check SETUP_GUIDE.md for detailed instructions!**

