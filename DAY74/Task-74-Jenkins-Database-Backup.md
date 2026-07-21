# 🌟 Task 74 - Automate MySQL Database Backup via Jenkins

**📌 Task Description**  
The **Nautilus DevOps team** needs to automate daily backups of the `kodekloud_db01` database from **Database Server** (`stdb01`) to **Backup Server** (`stbkp01`). A Jenkins job must run **every 10 minutes**, create a dated SQL dump, and securely transfer it.

**👉 Task Requirements**:  
- Access Jenkins UI and log in with **username**: `admin`, **password**: `Adm!n321`.  
- Create a **Freestyle project** job named `database-backup`.  
- Take `mysqldump` of `kodekloud_db01`:  
  - **User**: `kodekloud_roy`  
  - **Password**: `asdfgdsd`  
- Name dump: `db_$(date +%F).sql` → e.g., `db_2025-11-15.sql`  
- Copy dump to: `/home/clint/db_backups` on `stbkp01`  
- **Schedule**: `*/10 * * * *` (exact format)  
- Use SSH + SCP securely.
For this KodeKloud Jenkins task, you need a Jenkins freestyle job that:

Runs a MySQL dump on App Server 1 (stapp01)
Names the dump db_YYYY-MM-DD.sql
Copies it to Storage Server (ststor01)
Runs every 10 minutes using the exact cron schedule

Assuming the common xFusion users:

App Server 1: tony@stapp01
Storage Server: natasha@ststor01

---
1. Configure SSH access

The Jenkins server needs to SSH to stapp01:

From Jenkins: run below command
       
       ssh tony@stapp01 hostname
If the output is shown without password being asked then continue.
If the password is asked then configure ssh keys on Jenkins sevrer, Then copy the ssh key to app sever 1:

      ssh-keygen -t rsa -N ""
      ssh-copy-id tony@stapp01

Similarly from stapp01 run below command

      ssh natasha@ststor01 hostname 
      
If output is shown without password being asked continue else configure ssh keys and copy to storage server
  on stapp01:

            ssh-keygen -t rsa -N ""
            ssh-copy-id natasha@ststor01
      
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

## 🔹 Step 4: Create Jenkins Job `database-backup`

**Action**: Create a new Freestyle project.  
**Purpose**: Foundation for automated backup.

**Steps**:  
1. From dashboard, click **New Item**.  
2. Enter:  
   - **Item name**: `database-backup`  
   - **Type**: **Freestyle project**  
3. Click **OK**.

**Success Indicators**:  
- ✅ Job configuration page opens.

---

## 🔹 Step 5: Schedule Build Every 10 Minutes

**Action**: Configure cron trigger.  
**Purpose**: Automate backup frequency.

**Steps**:  
1. Check **Build periodically**.  
2. Enter cron schedule:  
```
   */10 * * * *
```  
   *(Runs every 10 minutes)*

**Success Indicators**:  
- ✅ Schedule saved correctly.

---

## 🔹 Step 6: Add Build Step - Execute Commands on stdb01

**Action**: Run `mysqldump`, install `sshpass`, and transfer file.  
**Purpose**: Create dump and copy to backup server.

**Steps**:  
1. Under **Build**, click **Add build step** → **Execute Shell Build Step**.  
2. In **Exec command**, enter:  
```bash
  ssh tony@stapp01 <<'EOF'
  mysqldump -ukodekloud_roy -pasdfgdsd kodekloud_db01 > /tmp/db_$(date +%F).sql
  scp /tmp/db_$(date +%F).sql natasha@ststor01:/home/natasha/db_backups/
  EOF
```  
4. Click **Save**.

**Success Indicators**:  
- ✅ Command saved.  
- ✅ No syntax errors.

---

## 🔹 Step 7: Trigger Build Manually

**Action**: Run job to test immediately.  
**Purpose**: Validate end-to-end functionality.

**Steps**:  
1. Go to `database-backup` job.  
2. Click **Build Now**.  
3. Monitor **Console Output** for:  
```
   Dumping database...
   Installing sshpass...
   db_2025-11-15.sql  100%
```

**Success Indicators**:  
- ✅ Build succeeds.  
- ✅ File transferred.

---

## 🔹 Step 8: Verify Backup on stbkp01

**Action**: SSH into storage server and check file.  
**Purpose**: Confirm dump was copied correctly.

**Steps**:  
```bash
ssh natasha@ststor01
```
```bash
ls -l /home/natasha/db_backups
```

**Expected Output**:  
```
-rw-r--r-- 1 clint clint 44958 Jul 22 12:16 db_2026-07-22.sql
```
```bash
head -20 db_2025-11-15.sql
```

**Expected Output**:  
```
-- MySQL dump 10.13  Distrib 8.0.30...
--
-- Host: localhost    Database: kodekloud_db01
```
```bash
exit
```

**Success Indicators**:  
- ✅ File exists with correct name.  
- ✅ SQL dump is valid.

---



---

## 📋 Quick Reference Guide

**Jenkins UI Steps**:  
1. **Login**: `admin` / `Adm!n321`  
2. **Install**: SSH Credentials, Publish Over SSH → Restart  
3. **Configure Publish Over SSH**:  
   - `stdb01`: peter / Sp!dy  
   - `stbkp01`: clint / H@wk3y3  
4. **Create job**: `database-backup` (Freestyle)  
5. **Schedule**: `*/10 * * * *`  
6. **Build Step** → Send files or execute commands over SSH (stdb01):  
```bash
   mkdir -p /tmp/db-backup
   mysqldump -u kodekloud_roy -p'asdfgdsd' kodekloud_db01 > /tmp/db-backup/db_$(date +%F).sql
   sudo apt install sshpass -y
   sshpass -p 'H@wk3y3' scp -o StrictHostKeyChecking=no /tmp/db-backup/*.sql clint@stbkp01:/home/clint/db_backups
   rm -rf /tmp/db-backup
```  
7. **Build Now** → Verify:  
```bash
   ssh clint@stbkp01 "ls -l /home/clint/db_backups/db_*.sql"
```

---

## 💡 Common SCP & MySQL Issues & Fixes

### **Issue 1: mysqldump: command not found**
**Problem**: MySQL client not installed  
**Fix**:  
```bash
sudo apt install mysql-client -y
```

### **Issue 2: Permission denied (publickey)**
**Problem**: SSH key auth fails  
**Fix**:  
- Use `sshpass` with password  
- Ensure `StrictHostKeyChecking=no`

### **Issue 3: Directory Not Found**
**Problem**: `/home/clint/db_backups` missing  
**Fix**:  
```bash
mkdir -p /home/clint/db_backups
chown clint:clint /home/clint/db_backups
```

---

## 🔧 Troubleshooting Job Failures

### **Error: Authentication Failed**
**Symptoms**: Permission denied  
**Solution**:  
- Double-check passwords: `Sp!dy`, `H@wk3y3`  
- Test manually:  
```bash
ssh peter@stdb01
```

### **Error: File Not Transferred**
**Symptoms**: No file on `stbkp01`  
**Solution**:  
- Check `sshpass` installation  
- Verify scp path and permissions

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered**:  
Securely backing up a remote MySQL database and transferring it to a backup server non-interactively every 10 minutes.

**💡 Solution Approach**:  
1. Used Publish Over SSH plugin  
2. Configured two SSH servers with password auth  
3. Created dated dump using `$(date +%F)`  
4. Used `sshpass` + `scp` for password-based transfer  
5. Cleaned up temp directory

**🎯 Key Success Factors**:  
- Correct SSH server names: `stdb01`, `stbkp01`  
- Working **Test Configuration**  
- `mysqldump` with inline password  
- `sshpass` for non-interactive SCP  
- Cron: `*/10 * * * *`

**⚠️ Critical Fixes**:  
- Used `sudo apt install sshpass -y`  
- Added `-o StrictHostKeyChecking=no`  
- Created `/tmp/db-backup` before dump  
- Removed temp files after transfer

**🔒 Best Practices Applied**:  
- Dated backups for versioning  
- Secure credential handling  
- Idempotent and safe commands  
- Scheduled automation  
- Full cleanup

**⚠️ Important Troubleshooting Concepts**:  
- Always **Test Configuration** in Publish Over SSH  
- Use **Build Now** before scheduling  
- Check **Console Output** for scp progress  
- Validate file integrity on backup server

---

## ⚠️ Important Production Notes

🔧 **Security**: Use SSH keys in production (not passwords)  
🔐 **Reliability**: Add email alerts on failure  
📊 **Scalability**: Support multiple databases  
🛡️ **Rotation**: Delete old backups (>30 days)  
📹 **Monitoring**: Track backup size and duration

---

## ✅ Task Completion Checklist

- [ ] Logged into Jenkins with `admin`/`Adm!n321`  
- [ ] Installed SSH Credentials Plugin and Publish Over SSH Plugin  
- [ ] Configured SSH servers: `stdb01` (peter) and `stbkp01` (clint)  
- [ ] Created job named `database-backup`  
- [ ] Configured cron schedule: `*/10 * * * *`  
- [ ] Added SSH build step with mysqldump and scp commands  
- [ ] Triggered manual build successfully  
- [ ] Verified backup file exists in `/home/clint/db_backups` on `stbkp01`  
- [ ] Documented all steps  

**🎉 Success Criteria Met When**:  
- Job builds successfully every 10 minutes  
- SQL dump is created with date format: `db_2025-11-15.sql`  
- File is transferred to backup server  
- Console output shows 100% transfer completion  
- Backup file is valid and readable
