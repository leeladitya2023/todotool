# Fix: Jenkins on Windows - Cannot Run "sh" Command

## ❌ Error:
```
Cannot run program "sh" (in directory "C:\ProgramData\Jenkins\.jenkins\workspace\toolredux"): CreateProcess error=2, The system cannot find the file specified
```

## 🔍 Problem:
Jenkins is running on **Windows**, but the Jenkinsfile uses **`sh`** commands which are for **Unix/Linux**. Windows doesn't have `sh` by default.

## ✅ Solution:

I've created a **cross-platform Jenkinsfile** that automatically detects Windows vs Linux and uses the correct commands:

- **Windows**: Uses `bat` commands
- **Linux/Unix**: Uses `sh` commands

---

## 🚀 Quick Fix:

### Step 1: Update Your Jenkinsfile

The `Jenkinsfile` has been updated to support both Windows and Linux automatically! ✅

It uses `isUnix()` to detect the platform:
```groovy
if (isUnix()) {
    sh 'npm ci'  // Linux/Mac
} else {
    bat 'call npm ci'  // Windows
}
```

### Step 2: Commit and Push

```bash
cd REACT\11todoReduxToolkit
git add Jenkinsfile
git commit -m "Update Jenkinsfile for Windows support"
git push
```

### Step 3: Build Again

1. Go to Jenkins
2. Click **"Build Now"**
3. ✅ Should work now!

---

## 📋 Alternative: Use Windows-Only Jenkinsfile

If you want a Windows-specific version (simpler, but won't work on Linux):

### Step 1: Use Jenkinsfile.windows

1. **Copy** `Jenkinsfile.windows` to `Jenkinsfile`:
   ```bash
   copy Jenkinsfile.windows Jenkinsfile
   ```

2. **Or** rename in Jenkins:
   - Script Path: `REACT/11todoReduxToolkit/Jenkinsfile.windows`

3. **Commit**:
   ```bash
   git add Jenkinsfile
   git commit -m "Use Windows Jenkinsfile"
   git push
   ```

---

## 🔧 What Changed:

### Before (Linux-only):
```groovy
sh '''
    npm ci
    npm run build
'''
```

### After (Cross-platform):
```groovy
if (isUnix()) {
    sh '''
        npm ci
        npm run build
    '''
} else {
    bat '''
        call npm ci
        call npm run build
    '''
}
```

---

## 📝 Key Differences:

### Windows (bat):
- Uses `bat` instead of `sh`
- Uses `call` before npm commands
- Uses `%VARIABLE%` instead of `${VARIABLE}`
- Uses `if errorlevel 1` for error checking
- Uses `2>nul` to suppress errors
- Uses `findstr` instead of `grep`
- Uses `timeout /t 5` instead of `sleep 5`
- Uses `dir` instead of `ls`

### Linux/Unix (sh):
- Uses `sh` commands
- Uses `${VARIABLE}` for variables
- Uses `|| true` for error handling
- Uses `grep` for search
- Uses `sleep` for delays
- Uses `ls` for listing

---

## ✅ Verification:

After updating, your pipeline should:

1. ✅ Detect Windows automatically
2. ✅ Use `bat` commands on Windows
3. ✅ Build successfully
4. ✅ Create Docker image
5. ✅ Run container

---

## 🐛 Troubleshooting:

### Issue: Still getting "sh" error

**Solution:**
1. Make sure you committed the new Jenkinsfile
2. Make sure Jenkins is using the updated file
3. Check Script Path in Jenkins configuration

### Issue: npm not found on Windows

**Solution:**
1. Install Node.js on Windows: https://nodejs.org/
2. Add Node.js to PATH
3. Restart Jenkins service:
   ```cmd
   net stop Jenkins
   net start Jenkins
   ```

### Issue: Docker not found on Windows

**Solution:**
1. Install Docker Desktop for Windows: https://www.docker.com/products/docker-desktop
2. Make sure Docker Desktop is running
3. Test: `docker --version` in Command Prompt

### Issue: curl not found on Windows

**Solution:**
- Windows 10 1803+ has curl built-in
- Or install Git Bash which includes curl
- Or skip health check (it's optional)

---

## 📚 Files Created:

1. **Jenkinsfile** - Cross-platform (auto-detects Windows/Linux) ✅ **Use This**
2. **Jenkinsfile.windows** - Windows-only version (backup)

---

## ✅ After Fix:

Your pipeline will now work on **Windows Jenkins**! 🎉

The cross-platform Jenkinsfile automatically:
- Detects if running on Windows or Linux
- Uses the correct commands (`bat` or `sh`)
- Handles errors appropriately for each platform
- Works the same way regardless of OS

---

**Most important: The Jenkinsfile is now updated and supports Windows automatically!** ✅

