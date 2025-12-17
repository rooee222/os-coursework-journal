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
![uname command output](images/uname-a%20command.png)

this shows running Linux kernel version 6.8.0-50 on a 64-bit (x86_64) system. This is the standard kernel for Ubuntu 24.04 LTS.
The hostname is "Ubuntu-server" and I'm logged in as user "amuser".

**Command: `free -h`**
![free command output](images/free%20-h%20command.png)
**What this shows:** 
- Total RAM: 4.6GB allocated to this VM
- Currently using: 330MB (very light usage)
- Available memory: 4.2GB (plenty available)
- No swap space configured (0B)
- The system is running efficiently with minimal memory usage

**Command: `df -h`**
![df command output](images/df%20-h%20command.png)
**What this shows:**
- Main filesystem (`/dev/sda2`): 49GB total virtual disk
- Currently using: 2.7GB (6% usage)
- Available space: 44GB 
- Plenty of disk space remaining for applications and data

**Command: `ip addr`**
![ip addr command output](images/ip%20addr%20command.png)
**What this shows:**
- `lo` (loopback): Local interface at 127.0.0.1
- `enp0s3`: Main network interface (VirtualBox adapter)
- Interface state: UP (active and working)
- Network ready for SSH connections

**Command: `lsb_release -a`**
![lsb_release command output](images/lsb_release%20-a%20command.png)
**What this shows:**
- Confirmed running Ubuntu 24.04.3 LTS
- Codename: "Noble Numbat"
- LTS version with 5 years of security updates

### Complete Command Output Screenshot
Here is the full terminal session showing all commands executed:
![All commands output](images/all%20commands.png)

[← Back to Home](README.md) | [Next Week →](week2.md)
