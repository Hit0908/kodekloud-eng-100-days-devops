**🌟 Task 15 - Install and Configure Nginx with SSL on App Server 3**

**📌 Task Description**

The system admins need to prepare **App Server 3** for application deployment with the following requirements:

1. Install and configure **Nginx** on **App Server **
2. Move the self-signed SSL certificate and key from `/tmp/nautilus.crt` and `/tmp/nautilus.key` to appropriate locations and deploy them in Nginx
3. Create an `index.html` file under Nginx document root with content **"Welcome!"**
4. Verify accessibility from jump host using `curl -Ik https://<app-server-ip>/`

👉 **Your task:** Set up Nginx with SSL/TLS encryption using provided certificates and create a basic welcome page.

💡 **Note:** You can find the infrastructure details by clicking on the **Details of all Users and Servers** button on the top-right section of the page.

# Nginx HTTPS Configuration on App Server 1 (Stratos Datacenter)

## Project Overview

This project documents the steps required to prepare **App Server 1** in the **Stratos Datacenter** for application deployment.

The tasks include:

* Installing Nginx web server
* Configuring SSL using an existing self-signed certificate
* Deploying the SSL certificate and key
* Creating a default web page
* Testing HTTPS access from the jump host

---

# Prerequisites

Before starting, make sure:

* You have SSH access to the jump host.
* You can connect to App Server 1.
* The SSL certificate and key exist:

```
/tmp/nautilus.crt
/tmp/nautilus.key
```

* Required privileges are available using `sudo`.

---

# Step 1: Connect to App Server 1

From the jump host, connect to App Server 1.

```bash
ssh <app-server-user>@<app-server-hostname>
```

### Explanation

* `ssh` is used to securely connect to a remote Linux server.
* Replace `<app-server-user>` with the provided username.
* Replace `<app-server-hostname>` with the App Server 1 hostname.

Example:

```bash
ssh user@app-server-1
```

---

# Step 2: Install Nginx

## For RHEL/CentOS Based Systems

Update package information:

```bash
sudo yum update -y
```

Install Nginx:

```bash
sudo yum install nginx -y
```

## For Ubuntu/Debian Based Systems

Update package information:

```bash
sudo apt update
```

Install Nginx:

```bash
sudo apt install nginx -y
```

### Explanation

* `yum` and `apt` are package managers used to install software.
* `nginx` is the web server that will host the application.
* `-y` automatically confirms installation prompts.

---

# Step 3: Start and Enable Nginx Service

Enable Nginx to start automatically after reboot:

```bash
sudo systemctl enable nginx
```

Start Nginx:

```bash
sudo systemctl start nginx
```

Check Nginx status:

```bash
sudo systemctl status nginx
```

### Explanation

* `systemctl` manages Linux services.
* `enable` configures automatic startup.
* `start` runs the service immediately.
* `status` verifies whether the service is running.

---

# Step 4: Create SSL Certificate Directory

Create a dedicated directory for SSL files:

```bash
sudo mkdir -p /etc/nginx/ssl
```

### Explanation

* `/etc/nginx` stores Nginx configuration files.
* `/etc/nginx/ssl` stores SSL certificates securely.
* `mkdir -p` creates the directory even if parent directories do not exist.

---

# Step 5: Move SSL Certificate and Key

Move the certificate:

```bash
sudo mv /tmp/nautilus.crt /etc/nginx/ssl/
```

Move the private key:

```bash
sudo mv /tmp/nautilus.key /etc/nginx/ssl/
```

Set secure permissions on the private key:

```bash
sudo chmod 600 /etc/nginx/ssl/nautilus.key
```

### Explanation

* `mv` moves files from one location to another.
* SSL certificates should be stored outside temporary directories.
* The private key must have restricted permissions to prevent unauthorized access.

After moving, files should exist:

```
/etc/nginx/ssl/nautilus.crt
/etc/nginx/ssl/nautilus.key
```

---

# Step 6: Configure Nginx for HTTPS

Create an SSL configuration file:

```bash
sudo vi /etc/nginx/conf.d/ssl.conf
```

Add the following configuration:

```nginx
server {

    listen 443 ssl;

    server_name _;

    ssl_certificate /etc/nginx/ssl/nautilus.crt;

    ssl_certificate_key /etc/nginx/ssl/nautilus.key;


    root /usr/share/nginx/html;

    index index.html;


    location / {

        try_files $uri $uri/ =404;

    }

}
```

Save and exit.

### Explanation

Configuration details:

| Directive             | Purpose                           |
| --------------------- | --------------------------------- |
| `listen 443 ssl`      | Enables HTTPS on port 443         |
| `server_name _`       | Accepts requests for any hostname |
| `ssl_certificate`     | Location of SSL certificate       |
| `ssl_certificate_key` | Location of private key           |
| `root`                | Nginx website document root       |
| `index`               | Default webpage file              |
| `try_files`           | Handles file requests securely    |

---

# Step 7: Create Website Homepage

Create the HTML file:

```bash
sudo vi /usr/share/nginx/html/index.html
```

Add:

```html
Welcome!
```

Save the file.

### Explanation

* `/usr/share/nginx/html` is the default Nginx document root.
* `index.html` is displayed when users visit the website.

---

# Step 8: Validate Nginx Configuration

Run:

```bash
sudo nginx -t
```

Expected result:

```
syntax is ok
test is successful
```

### Explanation

This command checks:

* Configuration syntax
* SSL file paths
* Invalid directives

It prevents restarting Nginx with a broken configuration.

---

# Step 9: Restart Nginx

Apply the new configuration:

```bash
sudo systemctl restart nginx
```

Verify:

```bash
sudo systemctl status nginx
```

---

# Step 10: Test HTTPS Connection

Return to the jump host:

```bash
exit
```

Run:

```bash
curl -Ik https://<app-server-1-hostname>/
```

Example:

```bash
curl -Ik https://app-server-1
```

Expected output:

```
HTTP/1.1 200 OK
Server: nginx
```

### Explanation

* `curl` sends HTTP requests from the command line.
* `-I` fetches only HTTP headers.
* `-k` allows testing with self-signed certificates.

---

# Final Directory Structure

After completion:

```
/etc/nginx/
├── conf.d/
│   └── ssl.conf
│
├── ssl/
│   ├── nautilus.crt
│   └── nautilus.key
│
└── html/
    └── index.html
```

---

# Result

The App Server 1 is now ready for application deployment with:

✅ Nginx installed
✅ HTTPS enabled
✅ SSL certificate configured
✅ Secure key storage
✅ Default webpage deployed
✅ HTTPS connectivity verified

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenges Encountered:**

1. **SSL Configuration**: Properly configuring Nginx to use self-signed SSL certificates with correct file paths and permissions
2. **Certificate Management**: Moving SSL files to appropriate system locations while maintaining security
3. **Service Integration**: Ensuring Nginx starts successfully with SSL configuration and serves content properly

**💡 Solution Approach:**

1. **Systematic File Management**: Moved SSL certificate and private key to standard system locations (`/etc/pki/tls/certs/` and `/etc/pki/tls/private/`)
2. **Security-First Permissions**: Applied appropriate permissions (644 for certificate, 600 for private key) to maintain security
3. **Nginx SSL Configuration**: Enabled the SSL server block in `/etc/nginx/nginx.conf` with proper certificate paths
4. **Configuration Validation**: Used `nginx -t` to verify syntax before service restart
5. **Content Deployment**: Created the required index.html file in the correct document root location
6. **Comprehensive Testing**: Verified functionality both locally and from the Jump Host using curl with SSL options

**🎯 Key Success Factors:**
- **Proper SSL file placement** in standard system directories for organization and security
- **Correct Nginx configuration** with accurate certificate paths and SSL directives
- **Permission management** ensuring certificate accessibility while protecting private key
- **Syntax validation** before service restart to prevent configuration errors
- **Multi-point testing** from both local and remote hosts to ensure complete functionality

**⚠️ Critical Configuration Details:**
- **SSL server block** must be properly uncommented and configured in nginx.conf
- **Certificate paths** must exactly match the file locations in the configuration
- **Document root** must be set correctly for the index.html file to be served
- **Self-signed certificates** require the `-k` flag in curl for testing (insecure flag)

**🔒 Security Implementation:**
- **Private key protection** with 600 permissions (root access only)
- **Certificate accessibility** with 644 permissions for service access
- **Standard system locations** for SSL files following best practices
- **SSL/TLS encryption** for secure web communication

---

## ⚠️ Important Production Notes

🔧 **SSL Certificate Management**: In production, use certificates from trusted Certificate Authorities instead of self-signed certificates

🔐 **Security Hardening**: Implement additional SSL security measures like HSTS, OCSP stapling, and strong cipher suites

📊 **Monitoring**: Set up monitoring for SSL certificate expiration and renewal processes

🛡️ **Backup Strategy**: Regularly backup SSL certificates and private keys in secure locations
