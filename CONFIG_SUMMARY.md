# Quick Configuration Summary

## 🎯 Which Jenkinsfile to Use?

### Option 1: Jenkinsfile.local (Simplified - Uses Docker)
- ✅ **Ready to use - No configuration needed!**
- Builds Docker image and runs container locally
- Best for: Learning, local testing

### Option 2: Jenkinsfile.no-docker (No Docker)
- ✅ **Minimal configuration needed**
- Just builds the React app (no Docker)
- Best for: Simple CI/CD without containers

### Option 3: Jenkinsfile (Full - With Deployment)
- ⚠️ **Needs configuration**
- Builds, pushes to registry, deploys to server
- Best for: Production with deployment

---

## 📋 What to Fill In - Quick Reference

### For Jenkinsfile.local (Simplified):
```
✅ Nothing! Ready to use as-is.
```

### For Jenkinsfile.no-docker (No Docker):
```
✅ Fill only if you want to deploy:

DEPLOY_HOST = ''  // Leave empty if no deployment
DEPLOY_USER = 'deploy'  // Your server username
DEPLOY_PATH = '/var/www/todo-app'  // Where to deploy on server
DEPLOY_SSH_CREDENTIALS = 'ssh-credentials'  // Jenkins credential ID
```

### For Jenkinsfile (Full - With Deployment):
```
⚠️ MUST FILL:

1. Docker Registry:
   DOCKER_REGISTRY = 'docker.io'  // or 'registry.example.com'
   DOCKER_CREDENTIALS = 'docker-creds'  // Jenkins credential ID

2. Deployment Server:
   DEPLOY_HOST = 'your-server-ip.com'  // Your server
   DEPLOY_USER = 'ubuntu'  // Server username
   DEPLOY_SSH_CREDENTIALS = 'ssh-creds'  // Jenkins credential ID
```

---

## 🚀 Quick Start (No Docker)

**Use: Jenkinsfile.no-docker**

1. **Copy Jenkinsfile.no-docker to Jenkinsfile**:
   ```bash
   cp Jenkinsfile.no-docker Jenkinsfile
   ```

2. **Update path if needed** (line 49, 60, 73, 84):
   - If Jenkins workspace is in `REACT/11todoReduxToolkit/`: Remove `cd REACT/11todoReduxToolkit`
   - If Jenkins workspace is in repo root: Keep as-is

3. **Optional - Add deployment** (lines 13-16):
   ```groovy
   DEPLOY_HOST = 'your-server.com'  // Fill if you have a server
   DEPLOY_USER = 'ubuntu'
   DEPLOY_PATH = '/var/www/todo-app'
   DEPLOY_SSH_CREDENTIALS = 'ssh-credentials'
   ```

4. **That's it!** ✅

---

## 📍 Path Configuration

**Check where Jenkins checks out your code:**

- **If Jenkins job points to repo root** (Ecohub/):
  ```groovy
  cd REACT/11todoReduxToolkit
  npm ci
  ```

- **If Jenkins job points to project folder** (REACT/11todoReduxToolkit/):
  ```groovy
  npm ci  // No cd needed
  ```

---

## ✅ Minimum Configuration Example

### Jenkinsfile.no-docker (Simplest):
```groovy
environment {
    APP_NAME = 'todo-reduxtoolkit'
    BUILD_DIR = 'dist'
    NODE_VERSION = '18'
    
    // Leave empty if no deployment
    DEPLOY_HOST = ''
    DEPLOY_USER = 'deploy'
    DEPLOY_PATH = '/var/www/todo-app'
    DEPLOY_SSH_CREDENTIALS = 'ssh-credentials'
}
```

**For local testing without deployment:**
- Just make sure `DEPLOY_HOST = ''` (empty)
- Pipeline will build and archive artifacts only

---

## 📝 Step-by-Step Setup

1. **Choose your Jenkinsfile**:
   - No Docker? → Use `Jenkinsfile.no-docker`
   - Simple Docker? → Use `Jenkinsfile.local`
   - Full deployment? → Use `Jenkinsfile`

2. **Rename if needed**:
   ```bash
   cp Jenkinsfile.no-docker Jenkinsfile
   ```

3. **Check paths** (see Path Configuration above)

4. **Fill deployment info** (only if you have a server)

5. **Push to GitHub**:
   ```bash
   git add Jenkinsfile
   git commit -m "Add CI/CD pipeline"
   git push
   ```

6. **Configure Jenkins job**:
   - Script Path: `REACT/11todoReduxToolkit/Jenkinsfile`

7. **Build!** 🎉

---

**For detailed explanations, see: JENKINSFILE_CONFIG.md**

