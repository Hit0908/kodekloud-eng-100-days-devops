# 🌟 Task 78 - Conditional Jenkins Pipeline with Branch Parameter

**📌 Task Description**  

The development team of xFusionCorp Industries is working on to develop a new static website and they are planning to deploy the same on Nautilus App Server using Jenkins pipeline. They have shared their requirements with the DevOps team and accordingly we need to create a Jenkins pipeline job. Please find below more details about the task:



Click on the Jenkins button on the top bar to access the Jenkins UI. Login using username admin and password Adm!n321.


Similarly, click on the Gitea button on the top bar to access the Gitea UI. Login using username sarah and password Sarah_pass123. There under user sarah you will find a repository named web_app that is already cloned on App Server 1 under /var/www/html. sarah is a developer who is working on this repository.


Add a slave node named App Server 1. It should be labeled as stapp01 and its remote root directory should be /home/sarah/jenkins_agent (the repository is cloned under /var/www/html).


We have already cloned repository on App Server 1 under /var/www/html.


Apache is already installed on the app server and is running on port 8080.


Create a Jenkins pipeline job named nautilus-webapp-job (it must not be a Multibranch pipeline) and configure it to:


Add a string parameter named BRANCH.

It should conditionally deploy the code from web_app repository under /var/www/html on App Server 1, as this is the document root of the app server. The pipeline should have a single stage named Deploy ( which is case sensitive ) to accomplish the deployment.

The pipeline should be conditional, if the value master is passed to the BRANCH parameter then it must deploy the master branch, on the other hand if the value feature is passed to the BRANCH parameter then it must deploy the feature branch.

LB server is already configured. You should be able to see the latest changes you made by clicking on the App button. Please make sure the required content is loading on the main URL https://<LBR-URL> i.e there should not be a sub-directory like https://<LBR-URL>/web_app etc.

## 🔹 Step 1: Access Jenkins & Gitea UI

**Action**: Log in to both systems.  
**Purpose**: Verify access and repo structure.

**Steps**:  
1. **Jenkins**: `admin` / `Adm!n321`  
2. **Gitea**: `sarah` / `Sarah_pass123`  
3. Confirm: `sarah/web_app` has `master` and `feature` branches  

**Success Indicators**:  
- ✅ Both UIs accessible.  
- ✅ Branches visible.

---

## 🔹 Step 2: Install Required Plugins

**Action**: Install SSH Build Agents and Pipeline plugins.  
**Purpose**: Enable SSH nodes and parameterized pipelines.

**Steps**:  
1. **Manage Jenkins** → **Plugins** → **Available plugins**.  
2. Install:  
   - **SSH Build Agents**  
   - **Pipeline**  
3. **Restart Jenkins**.  
4. Re-login.

**Success Indicators**:  
- ✅ Plugins active.  
- ✅ Pipeline job type available.

---

## 🔹 Step 3: Prepare Storage Server (ststor01)

**Action**: Install Java, fix permissions.  
**Purpose**: Run Jenkins agent and allow file ops.

**Steps**:  
```bash
ssh sarah@stapp01  #ssh with sarah username as the homedirectory mentioned is /home/sarah/jenkins_agent and /var/www/html belongs to that user
```
```bash
# Install Java
sudo yum install java-17-openjdk -y  #jdk version 17 for this case, try to find exact supported versions for other cases
java -version
```
```bash
Verify whether the root directory mentione is accessible to user sarah

ls -ld /home/sarah/jenkins_agent
ls -ld /var/www/html

```
```bash
exit
```

**Success Indicators**:  
- ✅ Java 17 installed.  
- ✅ `/var/www/html` owned by sarah.

---

## 🔹 Step 4: Add SSH Credentials

**Action**: Store natasha credentials.  
**Purpose**: Secure SSH access.

**Steps**:  
1. **Manage Jenkins** → **Credentials** → **System** → **Global**.  
2. **Add Credentials** → **Username with password**.  
3. Enter:  
   - **Username**: `sarah`  
   - **Password**: `Sarah_pass123`  
   - **ID**: `stapp01`  
   - **Description**: `Application Server Access`  
4. Click **OK**.

**Success Indicators**:  
- ✅ Credential ID: `stapp01`.

---

## 🔹 Step 5: Add Storage Server as SSH Agent

**Action**: Register `stapp01` as Jenkins slave.  
**Purpose**: Run pipeline on correct host.

**Steps**:  
1. **Manage Jenkins** → **Nodes** → **New Node**.  
2. Enter:  
   - **Node name**: `App Server 1`  
   - **Type**: `Permanent Agent`  
3. Click **OK**.  
4. Configure:  
   - **Remote root directory**: `/home/sarah/jenkins_agent`  
   - **Labels**: `stapp01`  
   - **Launch method**: `Launch agent via SSH`  
   - **Host**: `stapp01`  
   - **Credentials**: `stapp01`  
   - **Host Key Verification Strategy**: `Non verifying Verification Strategy`  
5. Click **Save**.

**Success Indicators**:  
- ✅ Node online.

---

## 🔹 Step 6: Create Parameterized Pipeline Job

**Action**: Create `nautilus-webapp-job` with `BRANCH` parameter.  
**Purpose**: Enable conditional deployment.

**Steps**:  
1. **New Item** →  
   - **Name**: `nautilus-webapp-job`  
   - **Type**: `Pipeline`  
2. Click **OK**.  
3. Configure:  
   - Check **This project is parameterized**  
   - **Add Parameter** → **String Parameter**  
     - **Name**: `BRANCH`  
     - **Default Value**: `master`  
     - **Description**: `Branch to deploy (master or feature)`  
4. **Pipeline** → **Definition**: `Pipeline script`  
5. Paste script:  
```groovy
pipeline {
    agent { label 'stapp01' }

    parameters {
        string(name: 'BRANCH', defaultValue: 'master', description: 'Branch to deploy')
    }

    stages {
        stage('Deploy') {
            steps {
                sh """
                cd /var/www/html
                git checkout ${params.BRANCH}
                git pull origin ${params.BRANCH}
                """
            }
        }
    }
}

```

6. Click **Save**.

**Success Indicators**:  
- ✅ Job saved.  
- ✅ Parameter visible.

---

## 🔹 Step 7: Test Pipeline with master Branch

**Action**: Build with default master.  
**Purpose**: Verify standard deployment.

**Steps**:  
1. Go to `nautilus-webapp-job`.  
2. Click **Build with Parameters**.  
3. Keep `BRANCH = master` → **Build**.  
4. Check **Console Output**:  
```
   Cloning into '/tmp/web_app'...
   total X
   -rw-r--r-- 1 natasha natasha  ... index.html
```

**Success Indicators**:  
- ✅ Build succeeds.  
- ✅ Files copied to `/var/www/html`.

---

## 🔹 Step 8: Test with feature Branch

**Action**: Build with feature.  
**Purpose**: Confirm conditional logic.

**Steps**:  
1. **Build with Parameters**.  
2. Set `BRANCH = feature` → **Build**.  
3. Verify console shows:  
```
   Switched to a new branch 'feature'
```

**Success Indicators**:  
- ✅ feature branch cloned.

---

## 🔹 Step 9: Verify Final Website

**Action**: Access via App button.  
**Purpose**: Confirm content is live.

**Steps**:  
1. Click **App** button.  
2. URL: `https://<LBR-URL>`  
3. Confirm:  
   - ✅ "Welcome to xFusionCorp Industries!"  
   - ✅ No `/web_app` in path  
   - ✅ Content matches selected branch  

**Success Indicators**:  
- ✅ Site loads at root.  
- ✅ Correct version displayed.

---

## 🔹 Step 10: Document the Process

**Action**: Capture all configuration steps and verification results.  
**Purpose**: Audit and replication.

**Steps**:  
1. Document all configuration details:  
   - Jenkins & Gitea login  
   - Plugin installation  
   - Java & permissions  
   - Credentials  
   - Node configuration  
   - Parameter + pipeline script  
   - master & feature builds  
   - Final website  

**Success Indicator**:  
- ✅ Full documentation completed.

---

## 📋 Quick Reference Guide

**Pipeline Script**:  
```groovy
pipeline {
    agent { label 'stapp01' }

    parameters {
        string(name: 'BRANCH', defaultValue: 'master', description: 'Branch to deploy')
    }

    stages {
        stage('Deploy') {
            steps {
                sh """
                cd /var/www/html
                git checkout ${params.BRANCH}
                git pull origin ${params.BRANCH}
                """
            }
        }
    }
}

```

**Parameter**:  
- **Name**: `BRANCH`  
- **Default**: `master`  
- **Values**: `master`, `feature`

**Final URL**: `https://<LBR-URL>` → Root only

---

## 💡 Common Issues & Fixes

### **Issue 1: git clone Fails**
**Problem**: Repository not found  
**Fix**:  
- Confirm Gitea URL  
- Test manually:  
```bash
git clone -b master http://git.stratos.xfusioncorp.com/sarah/web_app.git
```

### **Issue 2: cp: cannot stat**
**Problem**: No files in `/tmp/web_app`  
**Fix**:  
- Add `ls -la /tmp/web_app` in script  
- Check clone output

### **Issue 3: Old Content Persists**
**Problem**: Previous files not removed  
**Fix**:  
- Use `rm -rf /var/www/html/*` before `cp` if needed

---

## 🔧 Troubleshooting Pipeline Failures

### **Error: sudo: no tty present**
**Symptoms**: sudo fails  
**Solution**:  
- Use `echo 'Bl@kW' | sudo -S ...`

### **Error: Branch Not Found**
**Symptoms**: `fatal: couldn't find remote ref feature`  
**Solution**:  
- Confirm branch exists in Gitea

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered**:  
Conditional deployment of two Git branches using a Jenkins parameter, with clean overwrite and no subfolder in final URL.

**💡 Solution Approach**:  
1. Used String Parameter `BRANCH`  
2. Cloned to temporary directory  
3. Used `sudo -S cp` to overwrite `/var/www/html`  
4. Cleaned up temp files  
5. Verified via LB root URL

**🎯 Key Success Factors**:  
- `BRANCH` parameter with default `master`  
- `git clone -b ${BRANCH}`  
- `sudo -S cp -r` with password  
- No subfolder in URL  
- Content: "Welcome to xFusionCorp Industries!"

**⚠️ Critical Fixes**:  
- `rm -rf /tmp/web_app` before/after  
- `sudo -S` for non-interactive  
- `http://git.stratos...` URL  
- Mount confirmed working

**🔒 Best Practices Applied**:  
- Parameterized builds  
- Temporary workspace  
- Idempotent deployment  
- Clean state  
- Full traceability

**⚠️ Important Troubleshooting Concepts**:  
- Always test Gitea URL manually  
- Use console logs to debug `sh`  
- Verify mount on app servers  
- Check Apache docroot

---

## ⚠️ Important Production Notes

🔧 **Security**: Use SSH keys + Gitea deploy keys  
🔐 **CI/CD**: Add Gitea webhook trigger  
📊 **Validation**: Add curl health check  
🛡️ **Rollback**: Keep last 3 versions  
📹 **Monitoring**: Alert on build failure

---

## ✅ Task Completion Checklist

- [ ] Logged into Jenkins with `admin`/`Adm!n321`  
- [ ] Logged into Gitea with `sarah`/`Sarah_pass123`  
- [ ] Verified `master` and `feature` branches exist  
- [ ] Installed SSH Build Agents and Pipeline plugins  
- [ ] Installed Java 21 on `ststor01`  
- [ ] Fixed ownership of `/var/www/html` to natasha  
- [ ] Added SSH credentials for `ststor01`  
- [ ] Created Storage Server node with label `ststor01`  
- [ ] Verified Storage Server node is **Online**  
- [ ] Created Pipeline job `nautilus-webapp-job`  
- [ ] Added String Parameter `BRANCH` with default `master`  
- [ ] Added single stage named `Deploy`  
- [ ] Configured pipeline with conditional `git clone -b ${BRANCH}`  
- [ ] Successfully built with `master` branch  
- [ ] Successfully built with `feature` branch  
- [ ] Verified website loads at `https://<LBR-URL>` (no subfolder)  
- [ ] Confirmed content shows "Welcome to xFusionCorp Industries!"  
- [ ] Documented all steps  

**🎉 Success Criteria Met When**:  
- Pipeline job accepts `BRANCH` parameter  
- Stage name is exactly `Deploy`  
- Pipeline runs on `ststor01` agent  
- Git clone uses `${BRANCH}` variable  
- Files deployed to `/var/www/html`  
- Website accessible via Load Balancer at root path  
- Both `master` and `feature` branches deploy successfully  
- Correct content displayed based on selected branch

