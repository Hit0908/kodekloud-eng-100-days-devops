# 🌟 Task 91 - Deploy httpd + Prepend Line to index.html Using Ansible

**📌 Task Description**  
The **Nautilus DevOps team** wants to:
1. Install and start **httpd** on all **3 App Servers**
2. Create `/var/www/html/index.html` with:  
```
   This is a Nautilus sample file, created using Ansible!
```
3. **Prepend** (add at top) using **lineinfile**:  
```
   Welcome to xFusionCorp Industries!
```
4. Set **owner**: `apache`, **group**: `apache`, **mode**: `0744`

**Validation Command**:  
```bash
ansible-playbook -i inventory playbook.yml
```

---

## 📋 Requirements

| **Item** | **Value** |
|----------|-----------|
| Inventory | `/home/thor/ansible/inventory` (exists) |
| Playbook | `/home/thor/ansible/playbook.yml` |
| Package | `httpd` |
| Service | `httpd` → started + enabled |
| File | `/var/www/html/index.html` |
| Owner/Group | `apache:apache` |
| Permissions | `0744` |
| First Line | `Welcome to xFusionCorp Industries!` |
| Second Line | `This is a Nautilus sample file...` |

---

## 📝 Solution Overview

### **Key Strategy**
- Use `copy` module → create file with initial content
- Use `lineinfile` with `insertbefore: BOF` → add line at top
- Apply ownership & mode on both tasks
- Run on `hosts: all`

---

## 🔹 Implementation Steps

### **Step 1: Connect to Jump Host**
```bash
ssh thor@jumphost
```

---

### **Step 2: Navigate to Ansible Directory**
```bash
cd /home/thor/ansible
```

---

### **Step 3: Create Playbook**
```bash
vi playbook.yml
```

**Content**:  
```yaml
---
- name: Deploy httpd and configure index.html with line at top
  hosts: all
  become: yes
  tasks:
    - name: Install httpd package
      yum:
        name: httpd
        state: present

    - name: Start and enable httpd service
      service:
        name: httpd
        state: started
        enabled: yes

    - name: Create index.html with initial content
      copy:
        dest: /var/www/html/index.html
        content: |
          This is a Nautilus sample file, created using Ansible!
        owner: apache
        group: apache
        mode: "0744"

    - name: Add welcome line at the top of index.html
      lineinfile:
        path: /var/www/html/index.html
        line: "Welcome to xFusionCorp Industries!"
        insertbefore: BOF
        owner: apache
        group: apache
        mode: "0744"
```

**Save & Exit**: `Esc` → `:wq` → `Enter`

**Critical**:  
- ✅ `insertbefore: BOF` → Beginning Of File  
- ✅ `owner`, `group`, `mode` on both tasks  
- ✅ `content:` with `|` → multi-line
- when you want to manage one line at a time without replacing the entire file.

Example 1: Add a line if it doesn't exist
         
         - name: Add a DNS server
           lineinfile:
             path: /etc/resolv.conf
             line: "nameserver 8.8.8.8"

If the line is already present, Ansible does nothing. If it's missing, Ansible adds it.
---

### **Step 4: Run Playbook**
```bash
ansible-playbook -i inventory playbook.yml
```

**Expected Output**:  
```
TASK [Create index.html with initial content] **********************************
changed: [stapp01]
changed: [stapp02]
changed: [stapp03]

TASK [Add welcome line at the top of index.html] *******************************
changed: [stapp01]
changed: [stapp02]
changed: [stapp03]
```

---

### **Step 5: Verify**

#### **Service Status**
```bash
ansible all -i inventory -m shell -a "systemctl is-active httpd"
```

**Expected**:  
```
stapp01 | CHANGED | rc=0 >> active
stapp02 | CHANGED | rc=0 >> active
stapp03 | CHANGED | rc=0 >> active
```

#### **File Permissions & Ownership**
```bash
ansible all -i inventory -m shell -a "ls -l /var/www/html/index.html"
```

**Expected**:  
```
-rwxr--r-- 1 apache apache ... /var/www/html/index.html
```

#### **File Content (Line 1 = Welcome)**
```bash
ansible stapp01 -i inventory -m shell -a "cat /var/www/html/index.html"
```

**Expected**:  
```
Welcome to xFusionCorp Industries!
This is a Nautilus sample file, created using Ansible!
```

---

## 📊 Code Analysis

### **Playbook** (`/home/thor/ansible/playbook.yml`)
```yaml
- name: Add welcome line at the top of index.html
  lineinfile:
    path: /var/www/html/index.html
    line: "Welcome to xFusionCorp Industries!"
    insertbefore: BOF
    owner: apache
    group: apache
    mode: "0744"
```

| **Parameter** | **Value** | **Purpose** |
|---------------|-----------|-------------|
| `insertbefore: BOF` | Add at top | Required |
| `owner: apache` | Web server user | Correct |
| `mode: "0744"` | `-rwxr--r--` | As required |

---

## 🔍 Verification Steps
```bash
# Run
ansible-playbook -i inventory playbook.yml

# Verify service
ansible all -i inventory -m shell -a "systemctl is-active httpd"

# Verify file
ansible all -i inventory -m shell -a "ls -l /var/www/html/index.html"

# Verify content
ansible stapp01 -i inventory -m shell -a "head -2 /var/www/html/index.html"
```

---

## 📖 Quick Command Reference
```bash
cd /home/thor/ansible

# Run playbook
ansible-playbook -i inventory playbook.yml

# Verify
ansible all -i inventory -m shell -a "cat /var/www/html/index.html"
```

---

## 💡 Common Issues & Fixes

| **Issue** | **Fix** |
|-----------|---------|
| Line not at top | Use `insertbefore: BOF` |
| Wrong owner | Add `owner: apache` to both tasks |
| File missing | `copy` creates it |
| Service down | `enabled: yes` |

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge**:  
Add line at top using `lineinfile` + preserve ownership

**💡 Solution**:  
```yaml
lineinfile:
  insertbefore: BOF
  owner: apache
  mode: "0744"
```

**🎯 Key Success Factors**:  
- ✅ `insertbefore: BOF`  
- ✅ `owner`/`mode` on both modules  
- ✅ `copy` with `content:`  
- ✅ httpd running

---

## ⚠️ Important Production Notes

### **Best Practice**:  
- Use `template` for complex HTML  
- Add `validate: '%s -f %s'`  
- Use handlers for restart

### **Security**:  
- Firewall: http service  
- SELinux: httpd context

---

## ✅ Task Completion Checklist

- [ ] SSH'd into jump host as `thor`
- [ ] Navigated to `/home/thor/ansible/`
- [ ] Verified inventory exists with all 3 servers
- [ ] Created `playbook.yml` with 4 tasks
- [ ] Task 1: Install httpd using `yum` module
- [ ] Task 2: Start and enable httpd using `service` module
- [ ] Task 3: Create index.html using `copy` module with `content:`
- [ ] Task 4: Prepend line using `lineinfile` module
- [ ] Set `owner: apache` and `group: apache` in both file tasks
- [ ] Set `mode: "0744"` in both file tasks
- [ ] Used `insertbefore: BOF` in lineinfile task
- [ ] Ran `ansible-playbook -i inventory playbook.yml`
- [ ] Verified httpd service is active on all servers
- [ ] Verified file permissions are `0744`
- [ ] Verified file ownership is `apache:apache`
- [ ] Verified first line is "Welcome to xFusionCorp Industries!"
- [ ] Verified second line is "This is a Nautilus sample file..."
- [ ] Documented all steps

**🎉 Success Criteria Met When**:
- httpd installed, started, and enabled on all 3 servers
- File `/var/www/html/index.html` created
- File has permissions `0744` (`-rwxr--r--`)
- File owned by `apache:apache`
- First line is "Welcome to xFusionCorp Industries!"
- Second line is "This is a Nautilus sample file, created using Ansible!"
- `lineinfile` used with `insertbefore: BOF`
- Command runs without additional arguments
- All servers show successful task completion

---

## 🏁 Task Completion Summary

**Completed**:
- ✅ httpd deployed on all 3 App Servers
- ✅ index.html created with initial content
- ✅ Welcome line prepended to top of file
- ✅ Correct ownership (apache:apache)
- ✅ Correct permissions (0744)
- ✅ Service started and enabled

**Final Status**: Task 91 completed successfully!  
**Outcome**: Web server deployed with customized index.html on all 3 App Servers — ready for validation.

---

## 🎓 Learning Outcomes

- ✅ Using `copy` module with `content:` for inline text
- ✅ Using `lineinfile` with `insertbefore: BOF`
- ✅ Prepending lines to beginning of files
- ✅ Setting ownership and permissions on file modules
- ✅ Combining multiple modules for complete configuration
- ✅ `BOF` (Beginning Of File) directive

---

## 🚀 Next Steps

- Use `template` module for dynamic content
- Add handlers for service restarts
- Implement HTML validation
- Add firewall rules
- Configure virtual hosts

