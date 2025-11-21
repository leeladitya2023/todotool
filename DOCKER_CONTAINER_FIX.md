# Fix: Docker Container Name Conflict

## ❌ Error:
```
Error response from daemon: Conflict. The container name "/todo-reduxtoolkit-app" is already in use
```

## 🔍 Problem:

Docker doesn't allow two containers with the same name. When you run the pipeline again, the old container still exists (even if stopped), so Docker can't create a new one with the same name.

## ✅ Solution:

Use `docker rm -f` to force remove the old container before creating a new one.

### What Changed:

**Before:**
```bash
docker run -d --name todo-reduxtoolkit-app ...
```

**After:**
```bash
docker rm -f todo-reduxtoolkit-app  # Remove old container (if exists)
docker run -d --name todo-reduxtoolkit-app ...  # Create new container
```

## 🔧 What `docker rm -f` Does:

- **`docker rm`**: Removes container
- **`-f`**: Force flag - stops and removes container if running
- **`>nul 2>&1`**: Suppresses errors if container doesn't exist (Windows)
- **`2>/dev/null || true`**: Suppresses errors if container doesn't exist (Linux)

**Result**: Old container is removed, new one can be created

## 📝 Updated Jenkinsfile:

The "Run Container" stage now:
1. **Removes** old container if it exists (doesn't fail if missing)
2. **Creates** new container with fresh image

## ✅ Benefits:

- ✅ No more name conflicts
- ✅ Always uses latest image
- ✅ Stops old container automatically
- ✅ Doesn't fail if container doesn't exist

---

**After committing and pushing, the pipeline will handle container conflicts automatically!** 🎉

