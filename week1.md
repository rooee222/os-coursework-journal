# Week 1: System Planning and Distribution Selection

[← Back to Home](README.md) | [Next Week →](week2.md)
introduction
This phase focuses on planning the operating system environment used for this coursework. The aim of Phase 1 is to select appropriate systems,
design the system architecture, document the network configuration, and collect basic system information using command-line tools.
 
## 1. System Architecture Diagram

![System Architecture](images/architecture-week1.png)

**Architecture Description:**
- Windows laptop hosts two VirtualBox VMs
- Ubuntu Desktop VM (workstation for SSH access)
- Ubuntu Server VM (headless server)
- Connected via VirtualBox network
- SSH connection from workstation to server

## 2. Distribution Selection Justification

### Chosen Distribution: Ubuntu Server 24.04 LTS

I chose **Ubuntu Server 24.04 LTS (Noble)** for my server system because:
- **Long-term support:** 5 years of security updates
- **Latest stable release:** More recent kernel and packages
- **Extensive documentation:** Large community with lots of tutorials
- **Package management:** APT package manager is easy to use
- **Industry relevance:** Widely used in cloud environments
- **Beginner-friendly:** Good documentation for learning

### Alternative Distributions Considered

**Debian:**
- Very stable but has older packages
- Less beginner-friendly documentation
- Ubuntu has better community support for learning

**CentOS Stream:**
- Uses different package manager (yum/dnf)
- More enterprise-focused
- Steeper learning curve for beginners

Ubuntu Server provides the best balance of modern packages, stability, and learning resources for this coursework.

## 3. Workstation Configuration Decision

I'm using **Option A: Ubuntu Desktop VM** as my workstation.

**Rationale:**
- Complete Linux environment with built-in SSH client
- Can practice Linux commands on the workstation too
- Isolated from my Windows host system
- Can take VM snapshots before making changes
- No need to install extra software on Windows
- Authentic Linux administration experience

## 4. Network Configuration Documentation

### VirtualBox Network Settings

**Network Mode:** I configured both VMs to communicate with each other in VirtualBox.

**IP Configuration:**
- Ubuntu Desktop (Workstation): Acts as SSH client
- Ubuntu Server: Configured with static IP for SSH access

Both VMs can communicate with each other. I will SSH from the workstation to the server using the server's IP address.

## 5. System Specifications Using CLI

### Ubuntu Server Specifications

**Command: `uname -a`**
```bash
Linux Ubuntu-server 6.8.0-50-generic #91-Ubuntu SMP PREEMPT_DYNAMIC Tue Nov 18 14:14:30 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
```
**What this shows:** Running Linux kernel version 6.8.0 on a 64-bit system. This is the kernel for Ubuntu 24.04 LTS.

![uname output](images/uname-command.png)

---

**Command: `free -h`**
```bash
              total        used        free      shared  buff/cache   available
Mem:           4.6Gi       402Mi       4.2Gi       1.0Mi       204Mi       4.2Gi
Swap:             0B          0B          0B
```
**What this shows:** 
- Total RAM: 4.6GB (allocated in VirtualBox)
- Currently using: 402MB
- Available: 4.2GB
- Swap space: Not configured

![free command output](images/free-command.png)

---

**Command: `df -h`**
```bash
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           471M  1016K  470M   1% /run
/dev/sda2        49G  2.6G   44G   6% /
tmpfs           2.3G     0  2.3G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           471M   12K  471M   1% /run/user/1000
```
**What this shows:**
- 49GB virtual hard drive allocated
- 2.6GB used by Ubuntu Server installation
- 44GB available for applications and data
- Only 6% disk usage - plenty of space

![df command output](images/df-command.png)

---

**Command: `ip addr`**
```bash
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
    valid_lft forever preferred_lft forever

2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:f7:b8:fc brd ff:ff:ff:ff:ff:ff
    inet fe80::a00:27ff:fef7:b8fc/64 scope link
    valid_lft forever preferred_lft forever
```
**What this shows:**
- Interface `enp0s3` is my network adapter
- Connection is UP and working
- Network interface is configured and ready for SSH

![ip addr output](images/ip-addr-command.png)


**Command: `lsb_release -a`**
```bash
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 24.04.3 LTS
Release:        24.04
Codename:       noble
```
**What this shows:**
- Confirmed: Ubuntu 24.04 LTS
- Codename: Noble
- LTS version with long-term support

![lsb_release output](images/lsb-release-command.png)

[← Back to Home](README.md) | [Next Week →](week2.md)
