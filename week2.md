# Week 2: Security Planning and Testing Methodology

[← Previous Week](week1.md) | [Home](README.md) | [Next Week →](week3.md)

1. Performance Testing Plan
Remote Monitoring Methodology:
I will monitor my Ubuntu Server remotely from my Ubuntu Desktop workstation using SSH. All performance data will be collected by connecting via SSH and running monitoring commands on the server.
Testing Approach:

Connect to server via SSH from workstation
Run monitoring commands to collect baseline data (when server is idle)
Start an application or workload on the server
Run the same monitoring commands during the workload to collect performance data
Compare baseline vs. workload data to identify performance characteristics
Record data in tables for analysis

Monitoring Commands I Will Use:

.top - to monitor CPU and memory usage in real-time
.free -h - to check memory usage
.df -h - to monitor disk space
.iostat - to measure disk I/O performance
.ss or netstat - to monitor network connections


2. Security Configuration Checklist
SSH Hardening

 .Configure SSH key-based authentication
 .Disable password authentication
 .Disable root login via SSH
 .Configure SSH to only accept connections from workstation IP

Firewall Configuration

 .Enable UFW firewall
 .Configure firewall to allow SSH from workstation only
 .Set default deny policy for incoming connections
 .Verify firewall rules are active

Mandatory Access Control

 .Enable AppArmor or SELinux
 .Verify access control is in enforce mode
 .Check that profiles are loaded and active

Automatic Updates

 .Configure automatic security updates
 .Verify automatic updates are enabled
 .Test that updates run automatically

User Privilege Management

 .Create a non-root administrative user
 .Configure sudo access for the administrative user
 .Ensure root login is disabled

Network Security

 .Configure VirtualBox host-only network
 .Verify network isolation
 .Document network configuration


3. Threat Model
Threat 1: Unauthorized SSH Access
Description: An attacker attempts to gain access to the server by guessing SSH passwords or exploiting weak authentication.
Mitigation Strategies:

Use SSH key-based authentication only (no passwords)
Configure firewall to allow SSH only from the workstation IP address (192.168.56.10)
Install fail2ban to automatically block repeated failed login attempts
Monitor SSH access logs regularly


Threat 2: Privilege Escalation
Description: An attacker with basic user access tries to gain root privileges by exploiting system vulnerabilities or misconfigurations.
Mitigation Strategies:

Keep the system updated with automatic security updates
Use AppArmor or SELinux to restrict what applications can do
Configure sudo to require password authentication
Disable direct root login
Regularly audit system logs for suspicious activity


Threat 3: Unpatched Vulnerabilities
Description: Security vulnerabilities in the operating system or installed software that haven't been patched could be exploited by attackers.
Mitigation Strategies:

Enable automatic security updates to ensure patches are applied promptly
Regularly check for available updates manually
Run security audits using tools like Lynis (in Week 7)
Only install necessary software to minimize attack surface
Monitor security advisories for Ubuntu

## Week 2 Reflection
This week I planned my security configuration and performance testing approach. I created checklists for the security controls I'll implement in
Weeks 4-5 and identified three main security threats with mitigation strategies.

**Next Steps:** Select applications for performance testing in Week 3.
[← Previous Week](week1.md) | [Home](README.md) | [Next Week →](week3.md)
