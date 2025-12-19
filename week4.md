# Week 4: Initial System Configuration & Security Implementation

[← Previous Week](week3.md) | [Home](README.md) | [Next Week →](week5.md)

## Overview

In this phase, I configured foundational security controls on my Ubuntu Server. All configurations were performed remotely via SSH from my Ubuntu Desktop workstation.
I implemented SSH key-based authentication, configured the firewall, and created a non-root administrative user.

## 1. SSH Key-Based Authentication Configuration

### Generating SSH Key Pair

On my **Ubuntu Desktop (workstation)**, I generated an SSH key pair:
```bash
ssh-keygen -t rsa -b 4096 -C "amrita@workstation"
```

The key was saved in the default location `/home/[username]/.ssh/id_rsa`. I chose not to use a passphrase for this lab environment.

![Generating SSH key pair](images/generating%20ssh%20key%20pair.png)


### Copying Public Key to Server

I copied my public key to the server using:
```bash
ssh-copy-id amuser@[192.168.0.15]
```

This command added my public key to the server's `~/.ssh/authorized_keys` file.

![Copying public key to server](images/copying%20public%20key%20to%20server.png)

### Testing Key-Based Authentication

I tested the connection and successfully logged in without entering a password:
```bash
ssh amuser@[192.168.0.15]
```

![Testing key-based authentication](images/testing%20key%20based%20authentication.png)


### Disabling Password Authentication

**Before configuration:**

I created a backup of the SSH configuration file:
```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```

**Configuration changes:**

I edited `/etc/ssh/sshd_config`:
```bash
sudo nano /etc/ssh/sshd_config
```

Changed the following lines:
- `PasswordAuthentication no` (was `yes`)
- `PubkeyAuthentication yes` (was commented out)
 
![SSH config before and after](images/ssh%20config%20before%20and%20after.png)

**After configuration:**

I restarted the SSH service to apply changes:
```bash
sudo systemctl restart sshd
```
**Viewing configuration differences:**
```bash
diff /etc/ssh/sshd_config.backup /etc/ssh/sshd_config
```

![Configuration difference](images/configuration%20difference.png)


## 2. Firewall Configuration (UFW)

### Initial Firewall Status

Before configuration, the firewall was inactive:
```bash
sudo ufw status
```

Output: `Status: inactive`

![Firewall inactive](images/firewall%20inactive.png)


### Configuring Firewall Rules

**Set default policies:**
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```
**Allow SSH from workstation only:**

First, I identified my workstation IP address. Then I configured the firewall to allow SSH only from my workstation:
```bash
sudo ufw allow from [WORKSTATION_IP] to any port 22
```

**Enable firewall:**
```bash
sudo ufw enable
```

![UFW enable](images/ufw%20enable.png)

### Firewall Ruleset Documentation

**Complete firewall configuration:**
```bash
sudo ufw status numbered
```

**Firewall rules:**
```
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 22                         ALLOW IN    [192.168.0.17]
```

![UFW status](images/ufw%20status.png)

**Verbose firewall status:**
```bash
sudo ufw status verbose
```

This shows:
- Default incoming policy: deny
- Default outgoing policy: allow
- SSH allowed only from workstation IP

![UFW verbose status](images/ufw%20verbose%20status.png)


## 3. User and Privilege Management

### Creating Non-Root Administrative User

I created a new administrative user called `adminuser`:
```bash
sudo adduser adminuser
```

I set a password and accepted default values for user information.

![Create admin user](images/create%20admin%20user.png)


### Configuring Sudo Access

I added `adminuser` to the sudo group:
```bash
sudo usermod -aG sudo adminuser
```

**Verifying sudo privileges:**
```bash
sudo -l -U adminuser
```

This confirmed that `adminuser` can execute commands with sudo.

![Verify sudo access](images/verify%20sudo%20access.png)


### Testing Administrative User

I logged in as `adminuser` from my workstation:
```bash
ssh adminuser@[192.168.0.15]
```

Then tested sudo access:
```bash
sudo whoami
```

Output: `root`

This confirms `adminuser` has administrative privileges.

![Adminuser sudo test 1](images/adminuser%20sudo%20test%201.png)    ![Adminuser sudo test 2](images/adminuser%20sudo%20test%202.png)


## 4. SSH Access Evidence

Successfully established SSH connection from workstation to server:
```bash
ssh amuser@[192.168.0.15]
```

Connection established without password prompt, confirming key-based authentication is working.

![SSH connection success](images/ssh%20connection%20success.png)


## 5. Remote Administration Evidence

All commands were executed remotely via SSH from my workstation. Below are examples of remote administration:

**Checking current user:**
```bash
whoami
```
Output: `amuser`

**Checking hostname:**
```bash
hostname
```
Output: `Ubuntu-server`

**Checking network configuration:**
```bash
ip addr show enp0s3
```

**Checking SSH service status:**
```bash
sudo systemctl status sshd
```

**Checking firewall status:**
```bash
sudo ufw status
```

![Remote commands execution 1](images/remote%20commands%20execution%201.png)     ![Remote commands execution 2](images/remote%20commands%20exe%202.png)



## Security Configuration Summary

| Security Control | Status | Details |
|------------------|--------|---------|
| SSH Key-Based Auth | ✅ Configured | Password authentication disabled |
| Firewall (UFW) | ✅ Enabled | SSH allowed from workstation IP only |
| Non-Root Admin User | ✅ Created | `adminuser` with sudo privileges |
| Password Authentication | ✅ Disabled | Key-based authentication only |
| Remote Access | ✅ Working | All configs done via SSH |

## Week 4 Reflection

This week I successfully implemented foundational security controls on my server. All configuration was done remotely via SSH,
which simulates real-world server administration.

**Next Steps:** In Week 5, I will implement advanced security controls including AppArmor, fail2ban, and create monitoring scripts.

[← Previous Week](week3.md) | [Home](README.md) | [Next Week →](week5.md)
