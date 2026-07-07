**🌟 Task 14 - Fix Apache Service Unavailability and Ensure Running on Port 5003**

**📌 Task Description**

The monitoring system reported **Apache service unavailability** on one of the app servers in **Stratos Datacenter**. The requirement is to identify the faulty server, fix the Apache service, and ensure Apache is running on **port 5003** on all app servers.

👉 **Your task:** Identify the problematic server, resolve Apache issues, and configure all app servers to run Apache on port 5003.

💡 **Note:** It is not necessary to have any hosted content for this task - focus on service availability and port configuration.

---

## 🔹 Step 1: Identify the problematic server

**Test all app servers from Jump Host:**

```bash
curl -I http://stapp01:5003
curl -I http://stapp02:5003
curl -I http://stapp03:5003
```

**Purpose**: Quickly identify which app server has Apache service issues.

**Expected Results**:
- ✅ **Working servers**: HTTP response with status codes
- ❌ **Faulty server**: Connection refused or timeout

---

## 🔹 Step 2: Login to the faulty server for diagnosis

**On App Server 1 (login as tony):**

```bash
sudo su -
```

**Purpose**: Switch to root user for administrative troubleshooting tasks.

---

## 🔹 Step 3: Check Apache service detailed status

```bash
systemctl status httpd -l
```

**Purpose**: Check the detailed status of Apache service to identify specific failure reasons.

**Problem Output Example**:
```
● httpd.service - The Apache HTTP Server
   Loaded: loaded
   Active: failed (Result: exit-code)
   Process: 1234 ExecStart=/usr/sbin/httpd -DFOREGROUND (code=exited, status=1/FAILURE)
   Main PID: 1234 (code=exited, status=1/FAILURE)
   
Error: (98)Address already in use: AH00072: make_sock: could not bind to address [::]:8084
```

**Issue Found**: 🔍 **Apache failing due to bind error on port 8084**

---

## 🔹 Step 4: Identify process using the conflicting port

```bash
sudo dnf install net-tools -y #Installs all netstat tools
netstat -luntp | grep 8084

```

**Purpose**: Show which process is listening on the conflicting port.

**Problem Output Example**:
```
tcp    0    0 127.0.0.1:8084    0.0.0.0:*    LISTEN    757/sendmail
```

**Issue Found**: 🔍 **Sendmail is occupying port 8084 (PID 757)**

**Alternative commands**:
```bash
ss -luntp | grep 8084
lsof -i :8084
```

---

## 🔹 Step 5: Terminate the conflicting process

```bash
kill 757
```

**Purpose**: Kill the conflicting process to free the port for Apache.

**Verify port is freed**:
```bash
netstat -luntp | grep 8084
```

**Expected Result**: No output (port is now available)

**Status**: ✅ **Port 8084 is now free for Apache**

---

## 🔹 Step 6: Configure Apache to use port 5003

```bash
sudo vi /etc/httpd/conf/httpd.conf
```

**Purpose**: Modify Apache main configuration to listen on port 5003 instead of default ports.

**Configuration Changes Required**:

**Find and modify these lines:**
```apache
# Change from:
Listen 80
Listen 8084

# Change to:
Listen 5003
```

**Additional configuration file (if exists)**:
```bash
sudo vi /etc/httpd/conf.d/your-custom.conf
```

**Add or modify**:
```apache
Listen 5003
<VirtualHost *:5003>
    DocumentRoot /var/www/html
    ServerName localhost
</VirtualHost>
```

---

## 🔹 Step 7: Verify Apache configuration syntax

```bash
sudo httpd -t
```

**Purpose**: Test Apache configuration syntax before restarting the service.

**Expected Output**:
```
Syntax OK
```

**If errors found**: Review and fix configuration files before proceeding.

---

## 🔹 Step 8: Restart Apache service

```bash
systemctl restart httpd
systemctl status httpd
```

**Purpose**: Restart Apache service and verify it starts successfully with new port configuration.

**Expected Output**:
```
● httpd.service - The Apache HTTP Server
   Loaded: loaded
   Active: active (running)
   Main PID: 1456 (httpd)
   Process: 1455 ExecStart=/usr/sbin/httpd -DFOREGROUND (code=exited, status=0/SUCCESS)
```

**Status**: ✅ **Apache is now active and running**

---

## 🔹 Step 9: Verify Apache is listening on port 5003

```bash
netstat -luntp | grep 5003
```

**Purpose**: Confirm Apache is listening on the correct port 5003.

**Expected Output**:
```
tcp6    0    0 :::5003    :::*    LISTEN    1456/httpd
```

**Additional verification**:
```bash
ss -luntp | grep 5003
curl -I http://localhost:5003
```

---

## 🔹 Step 10: Test local connectivity

```bash
curl http://localhost:5003
```

**Purpose**: Test Apache response locally on the server.

**Expected Output**: Apache default page or basic HTTP response

**Status**: ✅ **Local connectivity confirmed**

---

## 🔹 Step 11: Check and configure other app servers

**Repeat Steps 2-10 on App Server 2 and App Server 3:**

```bash
# On stapp02 (login as steve)
sudo systemctl status httpd
sudo vi /etc/httpd/conf/httpd.conf
# (Change Listen directive to 5003)
sudo systemctl restart httpd

# On stapp03 (login as banner)  
sudo systemctl status httpd
sudo vi /etc/httpd/conf/httpd.conf
# (Change Listen directive to 5003)
sudo systemctl restart httpd
```

**Purpose**: Ensure all app servers are configured consistently to run Apache on port 5003.

---

## 🔹 Step 12: Final connectivity testing from Jump Host

**From Jump Host:**

```bash
curl http://stapp01:5003
curl http://stapp02:5003
curl http://stapp03:5003
```

**Purpose**: Test reachability from Jump Host for all app servers to confirm consistent configuration.

**Expected Results**: ✅ **All servers should return HTTP responses**

**HTTP test with headers**:
```bash
curl -I http://stapp01:5003
curl -I http://stapp02:5003
curl -I http://stapp03:5003
```

---

## 🔹 Step 13: Enable Apache service for auto-start

**On all app servers:**

```bash
sudo systemctl enable httpd
```

**Purpose**: Ensure Apache starts automatically on system boot.

**Verify service status**:
```bash
systemctl is-enabled httpd
systemctl list-unit-files | grep httpd
```

---

## 📋 Quick Command Reference

For the **problematic server** (identified through testing):

```bash
# Diagnosis and fix
sudo su -
systemctl status httpd -l
netstat -luntp | grep [conflicting_port]
kill [PID]

# Configuration change
sudo vi /etc/httpd/conf/httpd.conf
# (Change Listen directive to 5003)
sudo httpd -t
systemctl restart httpd
systemctl enable httpd
systemctl status httpd

# Verification
netstat -luntp | grep 5003
curl http://localhost:5003
```

**For all app servers:**
```bash
# From Jump Host - Test all servers
curl http://stapp01:5003
curl http://stapp02:5003  
curl http://stapp03:5003
```

---

## 💡 Additional Tips

- **Check all configuration files**: Look for additional Listen directives in `/etc/httpd/conf.d/*.conf`
- **Firewall considerations**: Ensure port 5003 is allowed if firewall is active
- **SELinux context**: May need to set appropriate SELinux context for non-standard ports
- **Log monitoring**: Check `/var/log/httpd/error_log` for detailed error information
- **Process verification**: Use `ps aux | grep httpd` to verify running processes

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Configuration syntax errors**
**Symptoms**: Apache fails to start after config changes
**Solution**: Use `httpd -t` to check syntax and fix errors
```bash
sudo httpd -t
sudo tail /var/log/httpd/error_log
```

### **Issue 2: SELinux blocking non-standard ports**
**Symptoms**: Service starts but connections are refused
**Solution**: Configure SELinux to allow Apache on port 5003
```bash
sudo semanage port -a -t http_port_t -p tcp 5003
sudo setsebool -P httpd_can_network_connect 1
```

### **Issue 3: Firewall blocking port 5003**
**Symptoms**: Local connections work but remote connections fail
**Solution**: Open port 5003 in firewall
```bash
sudo firewall-cmd --permanent --add-port=5003/tcp
sudo firewall-cmd --reload
```

### **Issue 4: Multiple Listen directives conflict**
**Symptoms**: Apache starts but listens on wrong ports
**Solution**: Review all configuration files for conflicting directives
```bash
sudo grep -r "Listen" /etc/httpd/conf*
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenges Encountered:**

1. **Service Identification**: Determining which app server had the Apache service failure among multiple servers
2. **Port Conflict Resolution**: Apache couldn't bind to port 8084 due to sendmail process occupying the port
3. **Configuration Standardization**: Ensuring all app servers consistently run Apache on port 5003

**💡 Solution Approach:**

1. **Systematic Testing**: Used curl commands from Jump Host to test all app servers and identify the problematic one
2. **Root Cause Analysis**: Used `systemctl status httpd -l` to get detailed error messages revealing the port bind failure
3. **Process Identification**: Used `netstat -luntp | grep 8084` to identify the sendmail process blocking the port
4. **Process Management**: Terminated the conflicting sendmail process to free the required port
5. **Configuration Update**: Modified `/etc/httpd/conf/httpd.conf` to change Listen directive from 8084 to 5003
6. **Consistency Implementation**: Applied the same port configuration across all app servers

**🎯 Key Success Factors:**
- **Detailed error analysis** using systemctl with `-l` flag for verbose output
- **Port conflict resolution** through process identification and termination
- **Configuration validation** using `httpd -t` before service restart
- **Comprehensive testing** from both local and remote hosts
- **Service persistence** through systemctl enable for automatic startup

**⚠️ Critical Learning Points:**
- **Port conflicts** are common causes of Apache startup failures
- **Multiple configuration files** may contain Listen directives that need coordination
- **Service availability testing** from external hosts is essential for validation
- **Process management** skills are crucial for resolving resource conflicts

**🔒 Production Considerations:**
- **Service monitoring** to detect failures quickly
- **Standardized configurations** across all app servers
- **Automated health checks** to verify service availability
- **Documentation** of custom port configurations for operations team

---

## ⚠️ Important Production Notes

🔧 **Multi-Server Consistency**: Ensure all app servers use the same port configuration for load balancing

🔐 **Security Considerations**: Non-standard ports may require additional firewall and SELinux configuration

📊 **Monitoring Setup**: Update monitoring systems to check port 5003 instead of default ports

🛡️ **Service Reliability**: Enable Apache service for automatic startup to prevent future unavailability
