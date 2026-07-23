# 🌟 Task 79 - Auto-Deploy Web App on Git Push via Jenkins

**📌 Task Description**  
The **Nautilus DevOps team** wants automatic deployment of the web app whenever developer **Sarah** pushes to the **Gitea master branch**. Jenkins must detect the push, pull the latest code, and deploy it to **Storage Server** at `/var/www/html`, which is shared across all **App Servers**.

**👉 Task Requirements**:  
- Access Jenkins UI and log in with **username**: `admin`, **password**: `Adm!n321`.  
- Access Gitea UI and log in with **username**: `sarah`, **password**: `Sarah_pass123`.  
- **Repo**: `sarah/web` → cloned under `/home/sarah/web` on Storage Server  
- **App Servers**: `stapp01`, `stapp02`, `stapp03`  
- **Apache**: Install `httpd`, serve on port **8080**  
- **Jenkins Job**: `nautilus-app-deployment`  
  - Freestyle project  
  - Git SCM: `http://git.stratos.xfusioncorp.com/sarah/web.git`  
  - Branch: `*/master`  
  - Trigger: Poll SCM → `* * * * *`  
  - Deploy: Copy all files to `/var/www/html` on App Server 1 
- **Ownership**: `/var/www/html` → `sarah:sarah`  
- **Final URL**: `https://<LBR-URL>` → No subfolder  
- **Content**: `Welcome to the xFusionCorp Industries`

---

## 🔹 PART 1: Install Apache on All App Servers

**Action**: Use Jenkins to install and configure httpd on port 8080.  
**Purpose**: Serve content from shared storage.

### **Step 1.1: Install Required Plugins**

**Steps**:  
1. **Manage Jenkins** → **Plugins** → **Available**.  
2. Install:  
   - **Publish Over SSH**  
   - **SSH Credentials**
   - **Git**
   - **SSH Build Agents**
3. **Restart Jenkins**.

**Success Indicators**:  
- ✅ Plugins active.


### **Step 1.4: Create apache-install-job**

## 🔹 PART 2: Configure Auto-Deployment Job

### **Step 2.2: Set Up Passwordless SSH (Jenkins → Storage Server)**

**On Jenkins Server**:  
```bash
ssh jenkins@jenkins
sudo su 
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""
cat ~/.ssh/id_rsa.pub
#copy the public key
```

**On App Server**:  
```bash
ssh sarah@stapp01
sudo su 
vi /.ssh/authorized_keys
# Paste Jenkins public key
chmod 600 /.ssh/authorized_keys
```

**Test**:  
```bash
ssh sarah@stapp01
**Success Indicators**:  
- ✅ No password prompt.

---

### **Step 2.3: Fix /var/www/html Ownership**

**On Storage Server**:  
```bash
ssh sarah@stapp01
sudo chown -R sarah:sarah /var/www/html
ls -ld /var/www/html
```

**Expected**:  
```
drwxr-xr-x 3 sarah sarah ... /var/www/html
```

---

### **Step 2.4: Create nautilus-app-deployment Job**

**Steps**:  
1. **New Item** → `nautilus-app-deployment` → **Freestyle project**.  
2. **Source Code Management** → **Git**:  
   - **Repository URL**: `http://git.stratos.xfusioncorp.com/sarah/web.git`  
   - **Credentials**: `None`  
   - **Branches**: `*/master`  
3. **Build Triggers** → **Poll SCM**:  
   - **Schedule**: `* * * * *`  
4. **Build** → **Add build step** → **Execute shell**:  
```bash
   echo "Deploying to Storage Server..."
   scp -o StrictHostKeyChecking=no \
       -i /var/lib/jenkins/.ssh/id_rsa \
       * sarah@stapp01:/var/www/html/
   echo "Deployment complete."
```  
5. **Save**.

**Success Indicators**:  
- ✅ Job configured.

---

## 🔹 PART 3: Push Change & Verify Auto-Deployment

### **Step 3.1: Update index.html and Push**

**On App Server**:  
```bash
ssh sarah@stapp01
cd /home/sarah/web
vi index.html
Paste this - Welcome to the xFusionCorp Industries

git add .
git commit -m "Updated index.html"
git push origin master
# Enter: sarah / Sarah_pass123
```

**Success Indicators**:  
- ✅ Push successful.

---

### **Step 3.2: Jenkins Auto-Triggers**

Wait **1–2 minutes** → Jenkins polls Gitea → detects change → builds.

**Console Output**:  
```
Started by an SCM change
...
scp ... index.html sarah@ststor01:/var/www/html/
Deployment complete.
Finished: SUCCESS
```

---

### **Step 3.3: Verify Final Website**

**Action**: Click **App** button  
**URL**: `https://<LBR-URL>`  
**Content**:  
```
Welcome to the xFusionCorp Industries
```

**Success Indicators**:  
- ✅ No subfolder.  
- ✅ All files deployed.  
- ✅ LB serves correct content.

---

## 📋 Quick Reference Guide

**Apache Install Script (per server)**:  
```bash
echo 'PASS' | sudo -S yum install -y httpd
echo 'PASS' | sudo -S sed -i 's/Listen 80/Listen 8080/' /etc/httpd/conf/httpd.conf
echo 'PASS' | sudo -S systemctl enable --now httpd
```

**Jenkins Deployment Command**:  
```bash
scp -o StrictHostKeyChecking=no -i /var/lib/jenkins/.ssh/id_rsa * sarah@ststor01:/var/www/html/
```

**Poll SCM Schedule**: `* * * * *` (every minute)

---

## 💡 Common Issues & Fixes

| **Issue** | **Fix** |
|-----------|---------|
| httpd not starting | Check Listen 8080 in config |
| SCP fails | Verify id_rsa path and permissions |
| Old content | Ensure scp overwrites all files |
| Poll SCM not triggering | Check Gitea URL and branch |

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge**:  
Auto-deploy on Git push with shared storage, passwordless SCP, and correct ownership.

**💡 Solution**:  
1. Apache on 8080 via Publish Over SSH  
2. Passwordless SSH using RSA key  
3. Poll SCM every minute  
4. SCP all files to `/var/www/html`  
5. Ownership: `sarah:sarah`

**🎯 Key Success Factors**:  
- `http://git.stratos.../sarah/web.git`  
- `* * * * *` polling  
- `scp -i /var/lib/jenkins/.ssh/id_rsa`  
- `chown -R sarah:sarah /var/www/html`  
- No subfolder in URL  
- Full repo deployed

---

## ⚠️ Important Production Notes

🔧 **Security**: Use Gitea webhooks instead of polling  
🔐 **Reliability**: Add post-deploy health check  
📊 **Scalability**: Use shared NFS for HA  
🛡️ **Monitoring**: Alert on failed builds  
📹 **Backup**: Git is source of truth

---

## ✅ Task Completion Checklist

- [ ] Logged into Jenkins with `admin`/`Adm!n321`  
- [ ] Logged into Gitea with `sarah`/`Sarah_pass123`  
- [ ] Installed Publish Over SSH and SSH Credentials plugins  
- [ ] Added credentials for all 3 app servers  
- [ ] Configured SSH servers in Publish Over SSH  
- [ ] Created and ran `apache-install-job`  
- [ ] Verified Apache running on port 8080 on all app servers  
- [ ] Installed Git and SSH Build Agents plugins  
- [ ] Set up passwordless SSH from Jenkins to Storage Server  
- [ ] Fixed ownership of `/var/www/html` to `sarah:sarah`  
- [ ] Created `nautilus-app-deployment` job  
- [ ] Configured Git SCM with correct repository URL  
- [ ] Set Poll SCM to `* * * * *`  
- [ ] Added SCP deployment command  
- [ ] Pushed changes to Gitea  
- [ ] Verified Jenkins auto-triggered build  
- [ ] Confirmed website loads at `https://<LBR-URL>`  
- [ ] Verified content shows "Welcome to the xFusionCorp Industries"  
- [ ] Documented all steps  

**🎉 Success Criteria Met When**:  
- Apache installed and running on port 8080 on all app servers  
- Jenkins job `nautilus-app-deployment` configured correctly  
- Poll SCM triggers build every minute  
- Git changes automatically deployed to `/var/www/html`  
- Website accessible via Load Balancer at root path  
- Content matches latest Git commit  
- No manual intervention required for deployment

