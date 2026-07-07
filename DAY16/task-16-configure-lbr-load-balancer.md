**🌟 Task 16 - Configure LBR Server as Nginx Load Balancer for App Servers**

**📌 Task Description**

To handle increasing website traffic on **Nautilus infrastructure** in **Stratos Datacenter**, the production support team decided to deploy a **high availability stack**. The **LBR (load balancer)** server configuration is the last pending task.

**Requirements:**
a. Install **Nginx** on **LBR server** (stlb01)
b. Configure **load balancing** in the main Nginx configuration file `/etc/nginx/nginx.conf` using **all App Servers**
c. Do not alter the **Apache port** on app servers and ensure **Apache service** is running
d. Verify **website access** through the **StaticApp button**

👉 **Your task:** Set up Nginx load balancer to distribute traffic across all app servers for high availability.

💡 **Note:** You can find the infrastructure details by clicking on the **Details of all Users and Servers** button on the top-right section of the page.

---

## 🔹 Step 1: Connect to LBR Server

```bash
ssh loki@stlb01
sudo su -
```

**Purpose**: Connect to the Load Balancer server (stlb01) as the loki user and switch to root for administrative tasks.

---

## 🔹 Step 2: Check Nginx service status (pre-installation)

```bash
systemctl status nginx
```

**Purpose**: Check if Nginx is already installed and running on the LBR server.

**Expected Output**:
```
Unit nginx.service could not be found.
```

**Status**: ❌ **Nginx is not installed yet** - proceeding with installation.

---

## 🔹 Step 3: Check OS version and install Nginx

```bash
yum install nginx -y
```

**Purpose**: Check the operating system version and install Nginx web server package.

**For Ubuntu/Debian systems**:
```bash
apt-get update
apt-get install nginx -y
```

**Installation Note**: 📦 This installs Nginx with default configuration suitable for load balancing.

---

## 🔹 Step 4: Start and verify Nginx service

```bash
systemctl start nginx
systemctl enable nginx
systemctl status nginx
```

**Purpose**: Start Nginx service, enable auto-start on boot, and verify it's running properly.

**Expected Output**:
```
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded
   Active: active (running)
   Main PID: 1234 (nginx)
```

---

## 🔹 Step 5: Check Apache ports on App Server 1

```bash
ssh tony@stapp01
sudo ss -luntp | grep httpd
```

**Purpose**: Connect to App Server 1 and check which port Apache is listening on.

**Expected Output Example**:
```
tcp    LISTEN  0    128    *:6300    *:*    users:(("httpd",pid=1234,fd=4))
```

**On each app server, confirm Apache is running:**
# On stapp01
```
ssh tony@stapp01 "sudo systemctl status httpd"

```
**Port Identified**: Apache is running on **port 6300**

**Alternative commands**:
```bash
sudo netstat -tlnp | grep httpd
ps aux | grep httpd
```

---

## 🔹 Step 6: Check Apache ports on App Server 2

```bash
ssh steve@stapp02
sudo ss -luntp | grep httpd

**On each app server, confirm Apache is running:**
# On stapp02

ssh tony@stapp02 "sudo systemctl status httpd"

```

**Purpose**: Connect to App Server 2 and verify Apache port configuration.

**Expected Result**: Apache should also be running on **port 6300** for consistency.

---

## 🔹 Step 7: Check Apache ports on App Server 3

```bash
ssh banner@stapp03
sudo ss -luntp | grep httpd

**On each app server, confirm Apache is running:**
# On stapp03
ssh tony@stapp03 "sudo systemctl status httpd"

```

**Purpose**: Connect to App Server 3 and confirm Apache port configuration.

**Expected Result**: Apache should be running on **port 6300** across all app servers.

**Port Summary**: 📊 All app servers should use the same Apache port (e.g., 6300).

**Purpose**: Ensure Apache service is active on all app servers before configuring load balancer.

**Expected Status**: ✅ **Active (running)** on all servers.

---

## 🔹 Step 9: Navigate to Nginx configuration directory

**Back on LBR server (stlb01):**

## 🔹 Step 10: Configure Nginx load balancing

```bash
sudo vi /etc/nginx/nginx.conf
```

**Purpose**: Edit the main Nginx configuration file to add load balancing configuration.

**Configuration Changes Required**:

**Add upstream configuration before the server block:**

```nginx
http {
    # Existing configuration...
    
    # Load Balancer Configuration
    upstream app_servers {
        server stapp01:6300;
        server stapp02:6300;
        server stapp03:6300;
    }
    
    # Default server configuration
    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;
        
        # Load balancer location
        location / {
            proxy_pass http://app_servers;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
        
        # Optional health check location
        location /nginx_status {
            stub_status on;
            access_log off;
            allow 127.0.0.1;
            deny all;
        }
    }
}
```

**Critical Configuration Points**:
- **Upstream block**: Defines backend app servers with their ports
- **Proxy pass**: Routes traffic to the upstream servers
- **Headers**: Preserves client information for backend servers
- **Load balancing method**: Default round-robin distribution

---

## 🔹 Step 11: Verify Nginx configuration syntax

```bash
nginx -t
```

**Purpose**: Test Nginx configuration syntax before applying changes.

**Expected Output**:
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

**If errors found**: Review and fix configuration syntax before proceeding.

---

## 🔹 Step 12: Restart Nginx with new configuration

```bash
systemctl restart nginx
systemctl status nginx
```

**Purpose**: Apply the new load balancer configuration and verify Nginx is running properly.

**Expected Status**: ✅ **Active (running)** with load balancer configuration loaded.

**Check listening ports**:
```bash
ss -tulnp | grep nginx
```

**Expected Output**:
```
tcp    LISTEN  0    128    *:80       *:*    users:(("nginx",pid=1234,fd=6))
```

---

## 🔹 Step 13: Test load balancer functionality

```bash
curl http://localhost/
curl http://stlb01/
```

**Purpose**: Test load balancer functionality locally and verify it's distributing traffic to app servers.

**Expected Response**: Content from one of the app servers (Apache response).

**Multiple requests test**:
```bash
for i in {1..3}; do curl -s http://localhost/ | head -1; done
```

---

## 🔹 Step 14: Verify backend server connectivity

```bash
# Test direct connectivity to each app server
curl http://stapp01:6300/
curl http://stapp02:6300/
curl http://stapp03:6300/
```

**Purpose**: Verify that all backend app servers are reachable and responding properly.

**Expected Result**: ✅ All app servers should return their respective content.

---

## 🔹 Step 15: Test through StaticApp button

**Using the web interface:**

1. **Click the StaticApp button** in the lab environment
2. **Verify the response** shows content from app servers
3. **Expected content**: "Welcome to xFusionCorp Industries!"

**Purpose**: Confirm that the load balancer is accessible through the intended interface and distributing traffic correctly.

**Success Indicators**:
- ✅ **StaticApp button loads successfully**
- ✅ **Content displays from backend servers**
- ✅ **Load balancer distributes requests**

---

## 🔹 Step 16: Monitor load balancer logs

```bash
tail -f /var/log/nginx/access.log
```

**Purpose**: Monitor real-time access logs to observe load balancing behavior.

**Log Analysis**: Look for requests being distributed among backend servers.

---

## 📋 Quick Command Reference

For quick copy-paste, execute on **LBR Server (stlb01)**:

```bash
# Connect and install Nginx
ssh loki@stlb01
sudo su -
systemctl status nginx
yum install nginx -y
systemctl start nginx && systemctl enable nginx

# Check app server ports
ssh tony@stapp01 "sudo ss -luntp | grep httpd"
ssh steve@stapp02 "sudo ss -luntp | grep httpd"
ssh banner@stapp03 "sudo ss -luntp | grep httpd"

# Configure load balancer
cd /etc/nginx/
vi nginx.conf
# (Add upstream and proxy_pass configuration)

# Test and restart
nginx -t
systemctl restart nginx
systemctl status nginx

# Test functionality
curl http://localhost/
# Use StaticApp button for final verification
```

---

## 💡 Additional Tips

- **Load balancing methods**: Add `ip_hash;` or `least_conn;` to upstream block for different algorithms
- **Health checks**: Configure `max_fails` and `fail_timeout` for server availability monitoring
- **SSL termination**: Configure HTTPS on load balancer if SSL is required
- **Monitoring**: Set up proper logging and monitoring for load balancer performance
- **Backup configuration**: Keep backup of working nginx.conf before making changes

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Backend servers not responding**
**Symptoms**: 502 Bad Gateway or connection errors
**Solution**: Verify app server status and port configuration
```bash
# Check app server status
systemctl status httpd
# Verify port listening
ss -tulnp | grep :6300
```

### **Issue 2: Configuration syntax errors**
**Symptoms**: `nginx -t` shows syntax errors
**Solution**: Review configuration file for missing brackets or semicolons
```bash
nginx -t
vi /etc/nginx/nginx.conf
# Fix syntax errors reported
```

### **Issue 3: Load balancer not distributing traffic**
**Symptoms**: All requests go to one server
**Solution**: Check upstream configuration and verify server availability
```bash
# Test each backend individually
curl http://stapp01:6300/
curl http://stapp02:6300/
curl http://stapp03:6300/
```

### **Issue 4: StaticApp button not working**
**Symptoms**: Web interface button doesn't load content
**Solution**: Check Nginx status and verify port 80 is accessible
```bash
systemctl status nginx
ss -tulnp | grep :80
curl http://stlb01/
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **configuring Nginx as a reverse proxy load balancer** to distribute incoming traffic across multiple Apache backend servers while maintaining service availability and proper request routing.

**💡 Solution Approach:**

1. **Backend Discovery**: Used `ss -luntp | grep httpd` on each app server to identify Apache listening ports (discovered port 6300)
2. **Service Verification**: Confirmed Apache service status across all app servers before load balancer configuration
3. **Upstream Configuration**: Created upstream block in nginx.conf with all three app servers and their respective ports
4. **Proxy Configuration**: Implemented proxy_pass directive to route traffic from port 80 to backend servers on port 6300
5. **Header Preservation**: Added proxy headers to maintain client information for backend servers
6. **Testing Strategy**: Used both local curl tests and StaticApp button verification to ensure proper functionality

**🎯 Key Success Factors:**
- **Port consistency verification** ensuring all app servers use the same Apache port (6300)
- **Proper upstream syntax** with correct server names and ports in the load balancer configuration
- **Header management** using proxy_set_header directives to preserve client information
- **Configuration validation** using `nginx -t` before applying changes to prevent service disruption
- **Multi-level testing** from local connectivity to web interface verification

**⚠️ Critical Configuration Details:**
- **Upstream block placement** must be within the http block but outside server blocks
- **Proxy pass syntax** requires `http://` prefix and must match upstream block name
- **Backend server ports** must be accurately determined and consistently configured
- **Load balancing algorithm** defaults to round-robin but can be customized for specific needs

**🔒 High Availability Benefits:**
- **Traffic distribution** across multiple backend servers prevents single point of failure
- **Automatic failover** when backend servers become unavailable
- **Scalability improvement** through horizontal load distribution
- **Performance optimization** by distributing client requests efficiently

---

## ⚠️ Important Production Notes

🔧 **High Availability**: Load balancer provides redundancy and improved performance across app servers

🔐 **Backend Health**: Implement proper health checks and monitoring for backend server availability

📊 **Traffic Distribution**: Monitor load balancing effectiveness and adjust algorithms as needed

🛡️ **Configuration Management**: Maintain consistent Apache port configuration across all app servers
