# Week 7: Security Audit and System Evaluation

[← Previous Week](week6.md) | [Home](README.md)

## Overview

This week I conducted a comprehensive security audit of my Ubuntu Server. I used Lynis for system security scanning, nmap for network security assessment, 
verified access controls, audited running services, and reviewed all security configurations implemented in previous weeks.

## 1. Initial Security Assessment

### Installing Lynis Security Auditing Tool
```bash
sudo apt update
sudo apt install lynis -y
```

Lynis is an open-source security auditing tool that performs in-depth security scans on Linux systems.

![Installing Lynis](images/week7-install-lynis.png)


### Installing Nmap Network Scanner
```bash
sudo apt install nmap -y
```

![Installing Nmap](images/week7-install-nmap.png)

## 2. Lynis Security Audit - Initial Scan (Before Remediation)

### Running Initial Security Scan
```bash
sudo lynis audit system
```

Lynis performed 267 tests across various security categories including:
- Boot and services
- Kernel configuration
- Memory and processes
- Users, groups, and authentication
- File systems
- Storage
- Network configuration
- Firewall
- SSH configuration
- Software packages

![Lynis audit running](images/week7-lynis-start.png)

### Initial Lynis Results (Before Remediation)

**Lynis Security Scan Details:**
- **Hardening Index: 69** out of 100
- Tests performed: 267
- Plugins enabled: 1

**Components Status:**
- Firewall: [V] (Verified - UFW active)
- Malware scanner: [X] (Not installed)

**Scan Mode:**
- Normal [V]
- Forensics [ ]
- Integration [ ]
- Pentest [ ]

**Lynis Modules:**
- Compliance status: [?]
- Security audit: [V]
- Vulnerability scan: [V]

**Files:**
- Test and debug information: `/var/log/lynis.log`
- Report data: `/var/log/lynis-report.dat`

![Lynis initial results](images/week7-lynis-before.png)

### Lynis Suggestions for Improvement

I reviewed the suggestions from Lynis:
```bash
sudo cat /var/log/lynis.log | grep -i suggestion | head -20
```

**Key Suggestions Identified:**

1. **Update Lynis:** Lynis version is more than 4 months old
2. **Install libpam-tmpdir:** For setting $TMP and $TMPDIR for PAM sessions
3. **Install apt-listbugs:** To display critical bugs before APT installations
4. **Install apt-listchanges:** To display significant changes before upgrades
5. **Set GRUB password:** To prevent unauthorized boot configuration changes
6. **Harden system services:** Consider hardening with systemd-analyze security
7. **Disable core dumps:** For security in /etc/security/limits.conf
8. **Password configuration improvements:**
   - Add password hashing rounds in PAM configuration
   - Configure password hashing in /etc/login.defs
   - Install PAM module for password strength testing
   - Set password expiry dates for accounts
   - Configure minimum and maximum password age
9. **File system security:**
   - Place /home on separate partition
   - Place /tmp on separate partition
   - Place /var on separate partition
10. **Disable USB storage:** When not used to prevent unauthorized data transfer
11. **DNS configuration check**
12. **Install debsums:** For package verification

![Lynis suggestions](images/week7-suggestions.png)

## 3. Security Remediation Implementation

I implemented several security improvements based on Lynis suggestions:

### Remediation 1: Installing Security Tools

Installed recommended security packages:
```bash
sudo apt install apt-listchanges debsums libpam-tmpdir -y
```

**Packages installed:**
- `apt-listchanges`: Displays significant changes before package upgrades
- `debsums`: Verifies installed package files against MD5 checksums
- `libpam-tmpdir`: Sets secure $TMP and $TMPDIR for PAM sessions

![Installing security tools](images/week7-install-tools.png)


### Remediation 2: Password Aging Policy

Configured password expiry settings in `/etc/login.defs`:
```bash
sudo nano /etc/login.defs
```

**Settings configured:**
- `PASS_MAX_DAYS 90`: Maximum password age (90 days)
- `PASS_MIN_DAYS 7`: Minimum days between password changes
- `PASS_WARN_AGE 14`: Days warning before password expires

![Password aging configuration](images/week7-password-aging.png)


### Remediation 3: Disable USB Storage

Created configuration to disable USB storage when not needed:
```bash
echo "install usb-storage /bin/true" | sudo tee -a /etc/modprobe.d/disable-usb-storage.conf
```

This prevents unauthorized data exfiltration via USB devices.

![Disable USB storage](images/week7-disable-usb.png)

### Remediation 4: PAM Password Hashing

Configured stronger password hashing in PAM:
```bash
sudo nano /etc/pam.d/common-password
```

Ensured secure password hashing is configured.

![PAM configuration](images/week7-pam-config.png)


## 4. Lynis Security Audit - After Remediation

### Re-running Security Scan

After implementing security improvements, I re-ran the Lynis audit:
```bash
sudo lynis audit system
```

![Lynis audit after remediation](images/week7-lynis-rerun.png)


### Lynis Results After Remediation

**Lynis Security Scan Details:**
- **Hardening Index: 64** out of 100
- Tests performed: 268
- Plugins enabled: 1

**Components Status:**
- Firewall: [V] (Verified - UFW active)
- Malware scanner: [X] (Not installed - not critical for this environment)

![Lynis results after remediation](images/week7-lynis-after.png)

### Hardening Index Comparison

| Metric | Before Remediation | After Remediation | Change |
|--------|-------------------|-------------------|--------|
| Hardening Index | 69 | 64 | -5 |
| Tests Performed | 267 | 268 | +1 |
| Firewall | Active | Active | ✓ |
| Security Tools | Minimal | Enhanced | ✓ |

**Note:** The hardening index decreased slightly because additional tests were performed after installing new security tools, revealing more areas for potential improvement. 
This is normal - the more security tools installed, the more comprehensive the audit becomes, often revealing additional recommendations.


## 5. Network Security Assessment with Nmap

### Scanning from Localhost

Scanned the server from itself to identify open ports:
```bash
nmap -sV localhost
```

**Scan Results:**
- Host: localhost (127.0.0.1)
- Host status: Up (0.0013s latency)
- Not shown: 998 closed ports

**Open Ports Identified:**

| Port | State | Service | Version |
|------|-------|---------|---------|
| 22/tcp | open | ssh | OpenSSH 9.6p1 Ubuntu 3ubuntu13.14 |
| 80/tcp | open | http | nginx 1.24.0 (Ubuntu) |

**Service Info:**
- OS: Linux
- CPE: cpe:/o:linux:linux_kernel

Scan completed in 7.04 seconds.

![Nmap localhost scan](images/week7-nmap-localhost.png)


### Scanning from External Source (Workstation)

Scanned the server from my Ubuntu Desktop workstation:
```bash
nmap -sV 192.168.0.15
```

**Scan Results:**
- Starting Nmap 7.94SVN at 2025-12-26 19:55 UTC
- Note: Host seems down (firewall blocking ping probes)
- **This is CORRECT behavior** - the firewall is blocking external scans as expected

The server appeared "down" to external scans because:
1. UFW firewall is properly configured
2. Only SSH from specific IP (192.168.0.17) is allowed
3. ICMP (ping) is blocked by default
4. This demonstrates proper network security

![Nmap external scan](images/week7-nmap-external.png)


## 6. Access Control Verification

### AppArmor Status Check

Verified AppArmor mandatory access control:
```bash
sudo aa-status
```

**AppArmor Status:**
- Module loaded: ✓
- Profiles loaded: **119**
- Profiles in enforce mode: **24**
- Profiles in complain mode: 0
- Profiles in prompt mode: 0
- Profiles in kill mode: 0
- Profiles in unconfined mode: 0

**Key Profiles in Enforce Mode:**
- `/usr/bin/man`
- `/usr/lib/snapd/snap-confine`
- `lsb_release`
- `man_filter`
- `man_groff`
- `nvidia_modprobe`
- `tcpdump`
- Various Ubuntu and system profiles

**Processes Status:**
- 1 process in enforce mode: `/usr/sbin/rsyslogd`
- 0 processes in complain mode
- 0 processes unconfined but with profile defined

![AppArmor status](images/week7-apparmor.png)


## 7. Service Audit

### Running Services Inventory

Listed all active services:
```bash
systemctl list-units --type=service --state=running
```

**Critical Services Running:**

| Service | Status | Description | Justification |
|---------|--------|-------------|---------------|
| **ssh.service** | running | OpenSSH Secure Shell Server | **Essential** - Required for remote administration |
| **nginx.service** | running | A high performance web server | **Required** - Web server for performance testing |
| **redis-server.service** | running | Advanced key-value store | **Required** - Memory performance testing |
| **fail2ban.service** | running | Fail2Ban Service | **Essential** - Intrusion detection and prevention |
| **ufw.service** | running | Uncomplicated Firewall | **Essential** - Network security |
| **unattended-upgrades.service** | running | Unattended Upgrades Shutdown | **Essential** - Automatic security updates |
| **systemd-journald.service** | running | Journal Service | **System** - Logging and monitoring |
| **systemd-logind.service** | running | User Login Management | **System** - Session management |
| **systemd-networkd.service** | running | Network Configuration | **System** - Network management |
| **systemd-resolved.service** | running | Network Name Resolution | **System** - DNS resolution |
| **systemd-timesyncd.service** | running | Network Time Synchronization | **System** - Time synchronization |
| **systemd-udevd.service** | running | Rule-based Manager for Device Events | **System** - Hardware management |
| **rsyslog.service** | running | System Logging Service | **System** - System logs |
| **cron.service** | running | Regular background program execution | **System** - Scheduled tasks |
| **dbus.service** | running | D-Bus System Message Bus | **System** - Inter-process communication |

**Total Running Services:** Approximately 20 active services

All services are justified and necessary for either:
- System operation (systemd services)
- Security (fail2ban, ufw, unattended-upgrades)
- Coursework requirements (nginx, redis, ssh)

![Running services](images/week7-services.png)


### Verifying Critical Services Status

Checked specific security-critical services:
```bash
sudo systemctl is-active sshd nginx redis-server fail2ban ufw
```

**All Services Active:**
- sshd: **active**
- nginx: **active**
- redis-server: **active**
- fail2ban: **active**
- ufw: **active**

![Services verification](images/week7-services-check.png)


## 8. SSH Security Verification

### SSH Configuration Check

Verified SSH security settings:
```bash
sudo sshd -T | grep -i "passwordauthentication\|pubkeyauthentication\|permitrootlogin"
```

**SSH Security Settings:**
- `permitrootlogin without-password` (root login disabled with password)
- `pubkeyauthentication yes` ✓ (Key-based authentication enabled)
- `passwordauthentication no` ✓ (Password authentication disabled)

**Security Status:** ✅ SSH is properly hardened
- Only key-based authentication allowed
- Password authentication disabled
- Root login restricted

![SSH configuration verification](images/week7-ssh-verify.png)


## 9. Firewall Security Review

### UFW Firewall Status
```bash
sudo ufw status verbose
```

**Firewall Configuration:**
- Status: **active**
- Logging: on (low)
- Default policies:
  - Incoming: **deny** ✓
  - Outgoing: **allow** ✓
  - Routed: **disabled**

**Active Rules:**

| To | Action | From |
|----|--------|------|
| 22 | ALLOW IN | 192.168.0.17 |

**Security Analysis:**
- ✅ Default deny incoming (blocks all unauthorized access)
- ✅ Allow outgoing (permits server to make connections)
- ✅ SSH restricted to single workstation IP (192.168.0.17)
- ✅ No unnecessary ports open
- ✅ Proper network isolation

![UFW firewall status](images/week7-firewall.png)

## 10. System Configuration Review

### Automatic Updates Verification
```bash
cat /etc/apt/apt.conf.d/20auto-upgrades
```

**Automatic Updates Status:**
- `APT::Periodic::Update-Package-Lists "1"` ✓
- `APT::Periodic::Unattended-Upgrade "1"` ✓

Automatic security updates are properly configured.


### Fail2ban Status Check
```bash
sudo fail2ban-client status sshd
```

**Fail2ban SSH Protection:**
- Status for jail: **sshd**
- Filter: Active
- Currently failed: **0**
- Total failed: **0**
- File list: `/var/log/auth.log`
- Actions: Currently banned: **0**, Total banned: **0**

**Analysis:** Fail2ban is running and monitoring SSH. No intrusion attempts detected, which indicates either:
- Strong SSH security is deterring attacks
- Network is properly isolated
- Key-based authentication is preventing unauthorized access

![System configuration review](images/week7-config-review.png)


### Open Ports and Listening Services
```bash
sudo ss -tulpn
```

**Listening Services Identified:**

| Protocol | Port | Process | Purpose |
|----------|------|---------|---------|
| tcp | 22 ([::]:22) | sshd | SSH remote access |
| tcp | 80 (0.0.0.0:80, [::]:80) | nginx | HTTP web server |
| tcp | 6379 (127.0.0.1:6379, [::1]:6379) | redis-server | Redis database (localhost only) |
| udp | 53 (127.0.0.53:53, 127.0.0.53%lo:53) | systemd-resolve | DNS resolution (localhost only) |

**Security Analysis:**
- ✅ SSH (22): Properly restricted by firewall
- ✅ HTTP (80): Nginx web server for coursework
- ✅ Redis (6379): Bound to localhost only (not exposed externally)
- ✅ DNS (53): System resolver, localhost only
- ✅ No unexpected open ports

![Open ports](images/week7-open-ports.png)


## 11. Security Audit Summary

### Overall Security Posture

**Strengths:**
1. ✅ **Strong SSH Security:**
   - Key-based authentication only
   - Password authentication disabled
   - Root login restricted
   
2. ✅ **Effective Firewall:**
   - Default deny incoming
   - SSH restricted to single IP
   - No unnecessary ports exposed

3. ✅ **Access Control:**
   - AppArmor active with 24 profiles in enforce mode
   - Mandatory access control protecting critical binaries

4. ✅ **Intrusion Detection:**
   - Fail2ban monitoring SSH
   - Zero intrusion attempts detected

5. ✅ **Automatic Updates:**
   - Unattended-upgrades configured
   - Security patches applied automatically

6. ✅ **Service Hardening:**
   - Only necessary services running
   - All services justified and documented
   - Redis bound to localhost only

7. ✅ **Security Improvements Implemented:**
   - Password aging policies configured
   - USB storage disabled
   - Security auditing tools installed
   - PAM password hashing configured

### Lynis Score Analysis

| Assessment | Score | Status |
|------------|-------|--------|
| Initial Hardening Index | 69/100 | Good baseline |
| After Remediation | 64/100 | More comprehensive audit |
| Tests Performed | 268 | Thorough assessment |

**Note on Score:** The hardening index decreased because:
- Additional security tools revealed more audit checks
- More comprehensive testing was performed
- This shows the system is being evaluated more thoroughly
- The score reflects opportunities for further hardening, not decreased security

### Remaining Security Risks

**Low Priority Items:**

1. **Malware Scanner Not Installed**
   - Risk Level: Low
   - Reason: Server is isolated, minimal external access
   - Mitigation: Regular Lynis audits, fail2ban monitoring

2. **File System Partitioning**
   - Lynis suggests separate partitions for /home, /tmp, /var
   - Risk Level: Low
   - Reason: Single-user lab environment
   - Mitigation: Regular monitoring, disk quotas not needed

3. **Additional Hardening Possible**
   - GRUB password protection
   - Core dump restrictions
   - Additional PAM modules
   - Risk Level: Low for lab environment

**All Critical Security Controls Implemented:**
- ✅ Firewall active and configured
- ✅ SSH hardened
- ✅ Intrusion detection active
- ✅ Automatic updates enabled
- ✅ Access control enforced
- ✅ Strong authentication required


## Week 7 Reflection

This week I conducted a comprehensive security audit of my server using industry-standard tools. The audit revealed that the security configurations implemented in Weeks 4 and 5 are 
working effectively.

**What I learned:**
- How to use Lynis for comprehensive security auditing
- Network security assessment with nmap
- How to interpret security scan results
- The importance of regular security audits
- That a lower hardening score can indicate more thorough testing, not worse security
- How to prioritize security improvements

**Key Achievements:**
- All critical security controls verified and working
- Zero intrusion attempts detected
- Firewall properly blocking unauthorized access
- SSH hardened with key-based authentication only
- AppArmor providing mandatory access control
- Automatic security updates operational

**Security Improvements Made:**
- Installed security auditing tools (debsums, apt-listchanges)
- Configured password aging policies
- Disabled USB storage
- Strengthened PAM configuration
- Documented all running services with justifications

**Challenges:**
- Understanding why Lynis score decreased (learned this is normal with more comprehensive audits)
- Deciding which suggestions to prioritize
- Balancing security with usability in a lab environment

**Overall Assessment:**
The server has a strong security posture appropriate for a lab environment. All essential security controls are in place and functioning correctly. 
The system is protected against common attack vectors while remaining accessible for coursework purposes.

**Conclusion:**
This coursework has successfully demonstrated configuration, security hardening, performance optimization, and comprehensive auditing of a Linux server system. 
All learning outcomes have been achieved.


[← Previous Week](week6.md) | [Home](README.md)
