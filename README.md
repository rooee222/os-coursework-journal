# os-coursework-journal
Operating Systems Coursework Technical Journal
student ID : A00020045
name       : Amrita Budha

 ## Project Overview
This journal documents my journey learning Linux server administration, security hardening, and performance optimization over 7 weeks.
I'm configuring an Ubuntu Server system accessible only via SSH from an Ubuntu Desktop workstation.

## Table of Contents
- [Week 1: System Planning and Distribution Selection](week1.md)
- [Week 2: Security Planning and Testing Methodology](week2.md)
- [Week 3: Application Selection](week3.md)
- [Week 4: Initial Configuration & Security](week4.md)
- [Week 5: Advanced Security and Monitoring](week5.md)
- [Week 6: Performance Evaluation](week6.md)
- [Week 7: Security Audit](week7.md)

## System Architecture

### Hardware Setup
- **Host Machine:** Windows Laptop
- **Virtualization:** Oracle VirtualBox

### Virtual Machines

**Server System:**
- **OS:** Ubuntu Server 24.04 LTS (Noble)
- **Type:** Headless (No GUI)
- **RAM:** 4.6 GB
- **Storage:** 49 GB virtual disk
- **Access:** SSH only
- **Hostname:** ubuntu-server
- **User:** amuser

**Workstation System:**
- **OS:** Ubuntu Desktop 22.04/24.04 LTS
- **Type:** Full desktop environment
- **Purpose:** Remote administration via SSH
- **Tools:** SSH client, monitoring utilities
- **Hostname:** [Your workstation hostname]

### Network Configuration
- **Network Type:** VirtualBox Network (configured for VM-to-VM communication)
- **Server Interface:** enp0s3
- **Connection:** SSH from workstation to server

