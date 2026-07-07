**🌟 Task 13 - Configure Firewall to Restrict Apache Port 8088 Access**

**📌 Task Description**

Due to security concerns, the **xFusionCorp Industries** team has decided to add a security layer by configuring **iptables firewalls** on all app servers in **Stratos Datacenter**.

**Requirements:**
1. Install **iptables** and all dependencies on all app servers
2. Block incoming traffic on **port 8088** (Apache port) on all app servers except from the **LBR host** (172.16.238.14)
3. Ensure the firewall rules **persist after system reboot**

👉 **Your task:** Configure selective port access using iptables to allow only the load balancer to reach Apache on port 8088.

💡 **Note:** Perform these steps on **all app servers**: stapp01, stapp02, stapp03.

---

## 🔹 Step 1: Switch to root user

```bash
sudo su -
```

**Purpose**: Switch to root user for administrative firewall configuration tasks.

**Note**: 🔐 Root privileges are required for iptables configuration and service management.

---

## 🔹 Step 2: Install iptables and dependencies

```bash
sudo dnf install -y iptables iptables-services
```

**Purpose**: Install iptables service package and all necessary dependencies.

**For Ubuntu/Debian systems**:
```bash
apt-get update
apt-get install -y iptables iptables-persistent
```

**Package Details**: 📦 This provides the iptables service management and persistence functionality.

---
```Enable and start the iptables service:

   sudo systemctl enable iptables
   sudo systemctl start iptables
```

## 🔹 Step 3: Clear existing iptables rules

```bash
iptables -F
```

**Purpose**: Flush (clear) all existing iptables rules to start with a clean configuration.

**Caution**: ⚠️ This removes all current firewall rules. Ensure you have console access in case of connection issues.

**Verify rules are cleared**:
```bash
iptables -L -n
```

## 🔹 Step 7: Allow port 8088 access from LBR host only

```bash
iptables -A INPUT -p tcp --dport 8088 -s <$LBRip> -j ACCEPT
```

**Purpose**: Allow Apache port 8088 access only from the Load Balancer (LBR) host IP.

**Rule Breakdown**:
- `-p tcp` - Protocol TCP
- `--dport 8088` - Destination port 8088
- `-s 172.16.238.14` - Source IP address (LBR host)
- `-j ACCEPT` - Allow this specific connection

**Security Focus**: 🎯 This implements the core security requirement.

---

## 🔹 Step 8: Drop all other traffic to port 8088

```bash
iptables -A INPUT -p tcp --dport 8088 -j DROP
```

**Purpose**: Block all other incoming connections to port 8088 (from any other source).

**Rule Breakdown**:
- `-p tcp` - Protocol TCP
- `--dport 8088` - Destination port 8088
- `-j DROP` - Silently drop the connection

**Security Note**: 🛡️ DROP is preferred over REJECT to avoid information disclosure.

---

## 🔹 Step 9: Add default reject rule

```bash
iptables -A INPUT -j REJECT --reject-with icmp-host-prohibited
```

**Purpose**: Reject all other incoming traffic not explicitly allowed by previous rules.

**Rule Breakdown**:
- `-j REJECT` - Reject the connection
- `--reject-with icmp-host-prohibited` - Send ICMP host prohibited response

**Default Policy**: 🚫 This creates a default deny policy for unmatched traffic.

---

## 🔹 Step 10: View configured rules

```bash
iptables -L -n --line-numbers
```

**Purpose**: Display all configured iptables rules with line numbers for verification.

**Expected Output**:
```
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination
1    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            
2    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
3    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:22
4    ACCEPT     tcp  --  172.16.238.14        0.0.0.0/0            tcp dpt:8088
5    DROP       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:8088
6    REJECT     all  --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited
```

---

## 🔹 Step 11: Save iptables rules

```bash
service iptables save
```

**Purpose**: Save the current iptables rules to make them persistent across reboots.

**Alternative commands**:
```bash
# For newer systems
systemctl save iptables
# Manual save
iptables-save > /etc/sysconfig/iptables
```

**Persistence**: 💾 Rules will survive system reboots.

---

## 🔹 Step 12: Start and enable iptables service

```bash
systemctl restart iptables
systemctl enable iptables
```

**Purpose**: Restart the iptables service and enable it to start automatically on boot.

**Service Status Check**:
```bash
systemctl status iptables
```

**Expected Status**: ✅ Active (exited) - Rules loaded successfully.

---

## 🔹 Step 13: Verify rules are active

```bash
iptables -L -n
```

**Purpose**: Confirm that all rules are properly loaded and active.

**Additional verification**:
```bash
# Check if port 8088 is listening
netstat -tlnp | grep 8088
# Test local connectivity
curl -I http://localhost:8088
```

---

## 🔹 Step 14: Verify from LBR host (stlb01)

**From Load Balancer host (stlb01):**

```bash
sudo telnet stapp01 8088
sudo telnet stapp02 8088
sudo telnet stapp03 8088
```

**Purpose**: Verify that connections to port 8088 are successful from the authorized LBR host.

**Expected Result**: ✅ All connections should be successful and show Apache response.

**Connection Success Output**:
```
Trying [IP_ADDRESS]...
Connected to stapp01.
Escape character is '^]'.
```

---

## 🔹 Step 15: Test blocking from unauthorized hosts

**From Jump Host or any other server (not LBR):**

```bash
telnet stapp01 8088
# Should timeout or be refused
```

**Purpose**: Confirm that connections from non-LBR hosts are properly blocked.

**Expected Result**: ❌ Connection should timeout or be refused.

---

## 📋 Quick Command Reference

For quick copy-paste, execute on **all app servers** (stapp01, stapp02, stapp03):

```bash
# Switch to root and install iptables
sudo su -

# install iptables and the iptables service
sudo yum install iptables iptables-services -y
                 or
sudo dnf install iptables iptables-services -y

# enable and start the iptables service
sudo systemctl enable iptables
sudo systemctl start iptables

# Add the needed iptables rules
sudo iptables -F  #removes all previous rules
sudo iptables -A INPUT -p tcp --dport 6400 -s 10.244.244.189 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 6400 -j REJECT

# save away the rules so the iptables service will find them on restart
sudo iptables-save > /etc/sysconfig/iptables


# Save and enable rules
service iptables save
systemctl restart iptables
systemctl enable iptables

# Verify configuration
iptables -L -n --line-numbers
```

**Verification from LBR host:**
```bash
# From stlb01 (LBR host)
sudo telnet stapp01 8088
sudo telnet stapp02 8088
sudo telnet stapp03 8088
```

---

## 💡 Additional Tips

- **Backup existing rules**: `iptables-save > /tmp/iptables_backup.txt` before making changes
- **Test connectivity**: Always test SSH access before saving rules
- **Rule modification**: Use `-D` to delete specific rules, `-I` to insert at specific position
- **Logging blocked attempts**: Add logging rules with `-j LOG --log-prefix "BLOCKED: "`
- **Service management**: Use `systemctl` commands for modern service control

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: SSH access lost after applying rules**
**Symptoms**: Cannot connect via SSH
**Prevention**: Always include SSH allow rule before saving
**Recovery**: Use console access to modify rules

### **Issue 2: Rules don't persist after reboot**
**Symptoms**: Rules disappear after server restart
**Solution**: Ensure iptables service is enabled and rules are saved
```bash
systemctl enable iptables
service iptables save
```

### **Issue 3: LBR host still cannot connect**
**Symptoms**: Authorized host connections fail
**Solution**: Verify LBR host IP address and rule order
```bash
# Check actual source IP from LBR
# Ensure ACCEPT rule comes before DROP rule
iptables -L -n --line-numbers
```

### **Issue 4: Service fails to start**
**Symptoms**: iptables service won't start
**Solution**: Check for syntax errors and conflicts
```bash
systemctl status iptables.service -l
journalctl -xeu iptables.service
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was implementing **selective access control** where only the Load Balancer (172.16.238.14) could access Apache on port 8088 while blocking all other traffic, including from management hosts.

**💡 Solution Approach:**

1. **Rule Ordering Strategy**: Configured rules in the correct sequence - ACCEPT before DROP to ensure proper traffic flow
2. **Source-based Filtering**: Used `-s 172.16.238.14` to specify the exact IP address allowed to access port 8088
3. **Default Deny Policy**: Implemented a comprehensive deny-all approach with specific exceptions
4. **Service Persistence**: Ensured rules survive reboots using `service iptables save` and `systemctl enable`

**🎯 Key Success Factors:**
- **Precise IP targeting**: Used the exact LBR host IP (172.16.238.14) for source-based access control
- **Proper rule sequence**: Placed ACCEPT rule before DROP rule to ensure authorized traffic is allowed
- **Essential service protection**: Maintained SSH access and loopback functionality
- **Persistence configuration**: Enabled automatic rule restoration after system reboots

**⚠️ Critical Configuration Details:**
- **Rule order matters**: The ACCEPT rule for 172.16.238.14 must come before the DROP rule for port 8088
- **Connection state handling**: ESTABLISHED,RELATED rule allows return traffic for outbound connections
- **Multiple server deployment**: Same configuration applied consistently across all app servers (stapp01-03)

**🔒 Security Benefits Achieved:**
- **Reduced attack surface**: Port 8088 is only accessible from the designated load balancer
- **Traffic segmentation**: Clear separation between management traffic and application traffic
- **Default deny posture**: All unspecified traffic is rejected by default

---

## ⚠️ Important Production Notes

🔧 **Multi-Server Deployment**: Apply identical rules across all app servers for consistent security posture

🔐 **Access Control**: Only the load balancer can reach Apache, improving security architecture

📊 **Monitoring**: Implement logging rules to track blocked access attempts for security analysis

🛡️ **Emergency Access**: Always maintain console access method in case of firewall lockout.
