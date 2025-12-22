# Week 5: Advanced Security and Monitoring Infrastructure

[← Previous Week](week4.md) | [Home](README.md) | [Next Week →](week6.md)


## Overview

This week I implemented advanced security controls and created monitoring scripts. I configured AppArmor for access control, enabled automatic security 
updates, set up fail2ban for intrusion detection, and created two scripts: one to verify security configurations and another to monitor server performance remotely.


## 1. AppArmor Access Control Implementation

Ubuntu Server comes with AppArmor enabled by default. I verified its status and documented how to track access control settings.

### Checking AppArmor Status
```bash
sudo aa-status
```

This command shows:
- How many profiles are loaded
- How many profiles are in enforce mode
- How many profiles are in complain mode
- Which applications have active profiles

![AppArmor status 1](images/apparmor%20status%201.png)

![AppArmor status 2](images/apparmor%20status%202.png)

![AppArmor status 3](images/apparmor%20status%203.png)


### AppArmor Configuration

**Verify AppArmor is enabled:**
```bash
sudo systemctl status apparmor
```

Output should show: `active (exited)`

![AppArmor configuration](images/apparmor%20configuration.png)

### Tracking Access Control Settings

**List all AppArmor profiles:**
```bash
sudo ls /etc/apparmor.d/
```

This shows all available security profiles on the system.

**Check which profiles are in enforce mode:**
```bash
sudo aa-status | grep "profiles are in enforce mode"
```

**View AppArmor logs for security events:**
```bash
sudo journalctl -u apparmor | tail -20
```

Or check system logs:
```bash
sudo grep -i apparmor /var/log/syslog | tail -20
```

![AppArmor logs](images/apparmor%20logs.png)

### AppArmor Reporting

**Generate AppArmor report showing all profiles:**
```bash
sudo aa-status --pretty-print
```

This provides a detailed view of:
- Profiles loaded and their mode (enforce/complain)
- Processes with profiles attached
- Processes in complain mode
- Processes without profiles

![AppArmor detailed report](images/apparmor%20detailed%20report.png)

## 2. Automatic Security Updates Configuration

I configured the system to automatically install security updates to keep the server protected from vulnerabilities.

### Installing Unattended-Upgrades
```bash
sudo apt update
sudo apt install unattended-upgrades -y
```
![Install unattended-upgrades](images/install%20unattended-upgrades.png)

### Enabling Automatic Updates
```bash
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

![Enabling automatic updates](images/enabling%20automatic%20updates.png)


### Configuring Automatic Updates

I edited the configuration file to ensure only security updates are installed automatically:
```bash
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

Verified that this line is uncommented:
```
"${distro_id}:${distro_codename}-security";
```

![Unattended-upgrades config](images/unattended-upgrades%20config.png)

### Verifying Automatic Updates Configuration

**Check if automatic updates are enabled:**
```bash
cat /etc/apt/apt.conf.d/20auto-upgrades
```

output:
```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

![Auto-updates verification](images/auto-updates%20verification.png)


### Testing Automatic Updates

**Run a dry-run to test the configuration:**
```bash
sudo unattended-upgrades --dry-run --debug
```

This simulates the update process without actually installing anything.

## 3. Fail2ban Configuration

Fail2ban monitors log files for suspicious activity (like repeated failed login attempts) and automatically blocks offending IP addresses.

### Installing Fail2ban
```bash
sudo apt update
sudo apt install fail2ban -y
```

![Installing fail2ban](images/installing%20fail2ban.png)


### Configuring Fail2ban for SSH

I created a local configuration file to protect SSH:
```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

**Configuration settings I verified/modified:**

Under the `[sshd]` section:
- `enabled = true` (enable SSH protection)
- `port = 22` (SSH port)
- `maxretry = 3` (ban after 3 failed attempts)
- `bantime = 600` (ban for 10 minutes)
- `findtime = 600` (time window for failed attempts)

![Fail2ban SSH config](images/fail2ban%20ssh%20config.png)

### Starting and Enabling Fail2ban
```bash
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
sudo systemctl status fail2ban
```

Output shows: `active (running)`

![Fail2ban status](images/fail2ban%20status.png)

### Verifying Fail2ban Configuration

**Check fail2ban status:**
```bash
sudo fail2ban-client status
```

**Check SSH jail specifically:**
```bash
sudo fail2ban-client status sshd
```

This shows:
- Number of failed login attempts
- Currently banned IP addresses
- Total number of bans

![Fail2ban client status](images/fail2ban%20client%20status.png)

## 4. Security Baseline Verification Script

I created a script that checks all security configurations from Weeks 4 and 5.

### Creating the Script

On the **server**, I created the script:
```bash
nano ~/security-baseline.sh
```

**Script content:**
```bash
#!/bin/bash
# Security Baseline Verification Script
# This script verifies all security configurations on the server
# Created for CMPN202 Operating Systems Coursework

echo "========================================"
echo "Security Baseline Verification Script"
echo "========================================"
echo ""

# Check 1: SSH Key-Based Authentication
echo "[1] Checking SSH Configuration..."
if grep -q "^PasswordAuthentication no" /etc/ssh/sshd_config; then
    echo "✓ Password authentication is disabled"
else
    echo "✗ WARNING: Password authentication is enabled"
fi

if grep -q "^PubkeyAuthentication yes" /etc/ssh/sshd_config; then
    echo "✓ Public key authentication is enabled"
else
    echo "✗ WARNING: Public key authentication is not enabled"
fi
echo ""

# Check 2: Firewall Status
echo "[2] Checking Firewall (UFW)..."
if sudo ufw status | grep -q "Status: active"; then
    echo "✓ Firewall is active"
    echo "Current rules:"
    sudo ufw status numbered
else
    echo "✗ WARNING: Firewall is not active"
fi
echo ""

# Check 3: AppArmor Status
echo "[3] Checking AppArmor..."
if sudo systemctl is-active --quiet apparmor; then
    echo "✓ AppArmor is running"
    ENFORCE_COUNT=$(sudo aa-status | grep "profiles are in enforce mode" | awk '{print $1}')
    echo "  Profiles in enforce mode: $ENFORCE_COUNT"
else
    echo "✗ WARNING: AppArmor is not running"
fi
echo ""

# Check 4: Automatic Updates
echo "[4] Checking Automatic Updates..."
if dpkg -l | grep -q unattended-upgrades; then
    echo "✓ Unattended-upgrades is installed"
    if grep -q "APT::Periodic::Unattended-Upgrade \"1\"" /etc/apt/apt.conf.d/20auto-upgrades; then
        echo "✓ Automatic updates are enabled"
    else
        echo "✗ WARNING: Automatic updates are not enabled"
    fi
else
    echo "✗ WARNING: Unattended-upgrades is not installed"
fi
echo ""

# Check 5: Fail2ban Status
echo "[5] Checking Fail2ban..."
if sudo systemctl is-active --quiet fail2ban; then
    echo "✓ Fail2ban is running"
    echo "SSH jail status:"
    sudo fail2ban-client status sshd | grep "Currently banned"
else
    echo "✗ WARNING: Fail2ban is not running"
fi
echo ""

# Check 6: Administrative User
echo "[6] Checking Administrative User..."
if id adminuser &>/dev/null; then
    echo "✓ Administrative user 'adminuser' exists"
    if groups adminuser | grep -q sudo; then
        echo "✓ adminuser is in sudo group"
    else
        echo "✗ WARNING: adminuser is not in sudo group"
    fi
else
    echo "✗ WARNING: adminuser does not exist"
fi
echo ""

# Summary
echo "========================================"
echo "Security Baseline Check Complete"
echo "========================================"
```

### Making the Script Executable
```bash
chmod 755 ~/security-baseline.sh
```

### Running the Security Baseline Script
```bash
./security-baseline.sh
```

![Security baseline script output](images/security%20baseline%20script%20output.png)

![Security baseline script output 1](images/security%20baseline%20script%20output%201.png)


## 5. Remote Monitoring Script

I created a script that runs on my **workstation** and collects performance data from the server via SSH.

### Creating the Script

On my **Ubuntu Desktop (workstation)**, I created:
```bash
nano ~/monitor-server.sh
```

**Script content:**
```bash
#!/bin/bash
# Remote Server Monitoring Script
# This script connects to the server via SSH and collects performance metrics
# Run this script from the workstation
# Created for CMPN202 Operating Systems Coursework

# Server connection details
SERVER_USER="amuser"
SERVER_IP="[192.168.0.15]"

echo "========================================"
echo "Remote Server Monitoring Script"
echo "Connecting to $SERVER_USER@$SERVER_IP"
echo "========================================"
echo ""

# Check 1: System Uptime and Load
echo "[1] System Uptime and Load Average:"
ssh $SERVER_USER@$SERVER_IP "uptime"
echo ""

# Check 2: CPU Usage
echo "[2] CPU Usage:"
ssh $SERVER_USER@$SERVER_IP "top -bn1 | grep 'Cpu(s)' | sed 's/.*, *\([0-9.]*\)%* id.*/\1/' | awk '{print \"CPU Usage: \" 100 - \$1 \"%\"}'"
echo ""

# Check 3: Memory Usage
echo "[3] Memory Usage:"
ssh $SERVER_USER@$SERVER_IP "free -h"
echo ""

# Check 4: Disk Usage
echo "[4] Disk Usage:"
ssh $SERVER_USER@$SERVER_IP "df -h | grep '^/dev/'"
echo ""

# Check 5: Network Connections
echo "[5] Active Network Connections:"
ssh $SERVER_USER@$SERVER_IP "ss -s"
echo ""

# Check 6: Running Services
echo "[6] Critical Services Status:"
echo "SSH Service:"
ssh $SERVER_USER@$SERVER_IP "sudo systemctl is-active sshd"
echo "Firewall Status:"
ssh $SERVER_USER@$SERVER_IP "sudo ufw status | head -1"
echo "Fail2ban Status:"
ssh $SERVER_USER@$SERVER_IP "sudo systemctl is-active fail2ban"
echo ""

# Check 7: Recent Failed Login Attempts
echo "[7] Recent Failed Login Attempts:"
ssh $SERVER_USER@$SERVER_IP "sudo grep 'Failed password' /var/log/auth.log | tail -5"
echo ""

echo "========================================"
echo "Monitoring Complete"
echo "========================================"
```

### Making the Monitoring Script Executable

On  **workstation**:
```bash
chmod 755 ~/monitor-server.sh
```

![Creating monitoring script and its output](images/creating%20monitoring%20script%20and%20its%20output.png)

### Running the Monitoring Script

On  **workstation**:
```bash
./monitor-server.sh
```

This script connects to the server via SSH and collects all performance metrics remotely.

![Monitoring script output 1](images/monitoring%20script%20output%201.png)

![Output 2](images/output%202.png)

![Output 3](images/output%203.png)

## Security Configuration Summary

| Security Control | Status | Verification Command |
|------------------|--------|---------------------|
| AppArmor | ✅ Enabled | `sudo aa-status` |
| Automatic Updates | ✅ Configured | `cat /etc/apt/apt.conf.d/20auto-upgrades` |
| Fail2ban | ✅ Running | `sudo fail2ban-client status sshd` |
| Security Baseline Script | ✅ Created | `./security-baseline.sh` |
| Monitoring Script | ✅ Created | `./monitor-server.sh` |


## Week 5 Reflection

This week I implemented advanced security controls and created automation scripts. The security baseline script helps verify all configurations quickly,
and the monitoring script allows me to check server performance remotely from my workstation.

**What I learned:**
- AppArmor provides mandatory access control without additional configuration
- Automatic updates help keep the system secure
- Fail2ban automatically protects against brute-force attacks
- Bash scripts can automate security checks and monitoring tasks

**Challenges:**
- Understanding AppArmor profile modes (enforce vs complain)
- Writing bash scripts with proper error checking
- Testing fail2ban without actually triggering bans

**Next Steps:** In Week 6, I will install the applications I selected in Week 3 and perform performance testing using the monitoring script I created this week.

---

[← Previous Week](week4.md) | [Home](README.md) | [Next Week →](week6.md)
