# 🌟 Task 73 - Copy Apache Logs from App Server 3 to Storage Server

**📌 Task Description**  
Click on the Jenkins button on the top bar to access the Jenkins UI. Login using username admin and password Adm!n321 1. Create a Jenkins jobs named copy-logs. 2. Configure it to periodically build every 9 minutes to copy the Apache logs (both access_log and error_log) from App Server 3 (stapp03) from the default logs location to location /usr/src/security on the Storage Server. 3. Build the job at least once so that the logs are copied and can be verified.

**👉 Task Requirements**:  
- Access Jenkins UI and log in with **username**: `admin`, **password**: `Adm!n321`.  
- Create a **Freestyle project** job named `copy-logs`.  
- Schedule build **every 10 minutes** using cron syntax.  
- Copy both Apache logs:  
  - `/var/log/httpd/access_log`  
  - `/var/log/httpd/error_log`  
- **Destination**: `/usr/src/security` on Storage Server (`ststor01`).  
- Use SSH + SCP via `banner@stapp03` and `natasha@ststor01`.  
- Verify logs are copied successfully.

---

## 🔹 Step 1: Access Jenkins UI

**Action**: Open Jenkins UI and log in with admin credentials.  
**Purpose**: Gain access to install plugins and configure job.

**Steps**:  
1. Click the **Jenkins** button in the top bar.  
2. Enter:  
   - **Username**: `admin`  
   - **Password**: `Adm!n321`  
3. Click **Log in**.

**Success Indicators**:  
- ✅ Login page loads successfully.  
- ✅ Dashboard appears after login.

---

## 🔹 Step 2: 

Ensure Jenkins can SSH to the source server (stapp03) if the job runs commands there.
Ensure the source server (stapp03) can SSH to the Storage Server if you're using scp/rsync from stapp03 to the storage server.

Generate an SSH key (if needed):

                ssh-keygen -t rsa -N ""

Copy the public key:

                ssh-copy-id banner@stapp03

Verify: Then SSH into stapp03 and configure passwordless SSH to the Storage Server:

          ssh-keygen -t rsa -N ""
          ssh-copy-id natasha@ststor01

verify: SSH into app server 3
           
           ssh banner@stapp03
           
and from stapp03: SSH into storage server

            ssh natasha@ststor01

Both should log in without prompting for a password.
               

If you're using the Jenkins Publish over SSH plugin instead, then only the Jenkins server needs SSH access to the destination server.

For the Jenkins job itself:
    
    New Item
    Job name: copy-logs
    Type: Freestyle project
    Build trigger: Build periodically

Schedule: A cronjob to move logs every 9 mins

                  */9 * * * *

Build step (Execute shell):

        ssh banner@stapp03 <<'EOF'
        scp /var/log/httpd/access_log /var/log/httpd/error_log natasha@ststor01:/usr/src/security/
        EOF
If Apache logs are under /var/log/apache2/ in your environment, replace the path accordingly. For xFusion labs, /var/log/httpd/ is the usual location.

or, if the job runs directly on stapp03:

cp /var/log/httpd/access_log /usr/src/security/
cp /var/log/httpd/error_log /usr/src/security/

(Use the first or second approach depending on your lab topology.)

Finally:

     Save the job.
     Click Build Now.

Verify on the Storage Server:

    ls -l /usr/src/security

You should see:

    access_log
    error_log
---

## 🔹 Step 10: Document the Process

**Action**: Capture all configuration steps and build execution.  
**Purpose**: Enable replication and audit.

**Steps**:  
1. Document all configuration details:  
   - Login credentials  
   - Plugin installation  
   - Credentials configuration (2)  
   - SSH hosts configuration (2)  
   - Job configuration (cron + SCP)  
   - Build console output  
   - Verification on `ststor01`  

**Success Indicator**:  
- ✅ All steps documented.

---

## 📋 Quick Reference Guide

**Jenkins UI Steps**:  
1. **Login**: `admin` / `Adm!n321`  
2. **Install**: SSH, SSH Credentials, Publish Over SSH → Restart  
3. **Add Credentials**:  
   - `stapp02-cred`: steve / Bl@kW  
   - `ststor01-cred`: natasha / Bl@kW  
4. **Add SSH Hosts**:  
   - `stapp02` → steve + stapp02-cred  
   - `ststor01` → natasha + ststor01-cred  
5. **Create job**: `copy-logs` (Freestyle)  
6. **Schedule**: `H/10 * * * *`  
7. **Build Step** → Execute shell on remote host (stapp02):  
```bash
   sshpass -p "Bl@kW" scp -p -o StrictHostKeyChecking=no /var/log/httpd/* natasha@ststor01:/usr/src/sysops
```  
8. **Build Now** → Verify on ststor01:  
```bash
   ssh natasha@ststor01 "ls -l /usr/src/sysops/"
```

---

## 💡 Common SCP & SSH Issues & Fixes

### **Issue 1: sshpass: command not found**
**Problem**: `sshpass` missing on `stapp02`  
**Fix**:  
```bash
sudo yum install -y sshpass
```

### **Issue 2: Permission Denied**
**Problem**: `natasha@ststor01: Permission denied`  
**Fix**:  
- Ensure `Bl@kW` is correct  
- Check `/usr/src/sysops` exists and is writable

### **Issue 3: Host Key Verification Failed**
**Problem**: Host key verification failed  
**Fix**:  
- Use `-o StrictHostKeyChecking=no`

---

## 🔧 Troubleshooting Job Failures

### **Error: Connection Refused**
**Symptoms**: Cannot connect to port 22  
**Solution**:  
- Verify host is up  
- Check firewall:  
```bash
ssh steve@stapp02 "sudo firewall-cmd --list-all"
```

### **Error: File Not Found**
**Symptoms**: `/var/log/httpd/*: No such file or directory`  
**Solution**:  
- Confirm Apache is running:  
```bash
ssh steve@stapp02 "ls -l /var/log/httpd/"
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered**:  
Securely copying logs from App Server 2 to Storage Server using non-interactive SSH and scheduled Jenkins job.

**💡 Solution Approach**:  
1. Installed SSH + Publish Over SSH plugins  
2. Stored credentials securely in Jenkins  
3. Configured SSH remote hosts with connection test  
4. Used `sshpass` + `scp` in remote shell on `stapp02`  
5. Scheduled job every 10 minutes

**🎯 Key Success Factors**:  
- Correct credential IDs: `stapp02-cred`, `ststor01-cred`  
- Working **Check Connection**  
- `sshpass` + `scp` with `-p` and `StrictHostKeyChecking=no`  
- Verified file transfer and content  
- Cron: `H/10 * * * *`

**⚠️ Critical Fixes**:  
- Used `sshpass -p "Bl@kW"` for non-interactive auth  
- Added `-o StrictHostKeyChecking=no`  
- Ensured `/usr/src/sysops` exists on `ststor01`

**🔒 Best Practices Applied**:  
- Centralized logging prep  
- Secure credential storage  
- Idempotent file transfer  
- Scheduled automation  
- Full audit trail

**⚠️ Important Troubleshooting Concepts**:  
- Test SSH manually first  
- Use **Build Now** before scheduling  
- Check **Console Output** for SCP progress  
- Validate destination path permissions

---

## ⚠️ Important Production Notes

🔧 **Security**: Never hardcode passwords — use Jenkins Credentials or SSH keys  
🔐 **Reliability**: Use `H/10` to avoid thundering herd  
📊 **Scalability**: Extend to multiple app servers  
🛡️ **Monitoring**: Add email notification on failure  
📹 **Backup**: Rotate logs in `/usr/src/sysops` periodically

---

## ✅ Task Completion Checklist

- [ ] Logged into Jenkins with `admin`/`Adm!n321`  
- [ ] Installed SSH Plugin, SSH Credentials Plugin, Publish Over SSH Plugin  
- [ ] Added credentials: `stapp02-cred` (steve) and `ststor01-cred` (natasha)  
- [ ] Configured SSH remote hosts: `stapp02` and `ststor01`  
- [ ] Created job named `copy-logs`  
- [ ] Configured cron schedule: `H/10 * * * *`  
- [ ] Added SSH build step with SCP command  
- [ ] Triggered manual build successfully  
- [ ] Verified logs exist in `/usr/src/sysops` on `ststor01`  
- [ ] Documented all steps  

**🎉 Success Criteria Met When**:  
- Job builds successfully every 10 minutes  
- Both `access_log` and `error_log` are copied  
- Files are readable on `ststor01`  
- Console output shows 100% transfer completion  
