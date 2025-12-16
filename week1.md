Phase 1: System Planning and Distribution Selection (Week 1)
intruduction
This phase focuses on planning the operating system environment used for this coursework. The aim of Phase 1 is to select appropriate systems, design the system architecture, document the network
configuration, and collect basic system information using command-line tools.

System Architecture Overview
This coursework uses a two-system architecture consisting of:
A server system running a Linux server operating system without a graphical interface
A workstation system used to manage the server remotely using SSH in later phases
This design reflects real-world server administration, where servers are managed remotely using command-line tools.

System Architecture Diagram
The architecture consists of a workstation connected to a headless Linux server through a private VirtualBox network. The server does not have a desktop environment and is accessed using command-line
tools only.


Distribution Selection Justification
I chose Ubuntu Server 22.04 LTS for my server system. I picked this because it's one of the most popular server distributions and has really good documentation online, which will help me when I get 
stuck. Ubuntu Server also has long-term support for 5 years, so it gets security updates for a long time.
I also considered Debian and CentOS Stream. Debian is very stable and lightweight, but Ubuntu has more recent packages and better community support for beginners. CentOS Stream looked interesting 
but it's more commonly used in enterprise environments and I found less beginner-friendly tutorials for it.
Ubuntu Server is perfect for learning because it has a large community, good package management with apt, and lots of guides available online.

 Workstation Configuration Decision
I'm using Option A - Ubuntu Desktop VM as my workstation. I'm running both VMs on my Windows laptop using VirtualBox.
I chose this option because having a separate Ubuntu Desktop VM means I have a proper Linux environment with all the tools I need already built in. It's easier to practice Linux commands on the
workstation too, and I don't need to install SSH clients or other tools on my Windows machine. Everything stays in the VMs, which keeps my main laptop clean. Also, if I mess something up, I can 
just restore the VM snapshot without affecting my laptop.

Network Configuration Documentation
I'm using VirtualBox Host-Only Network for both VMs to communicate with each other. This creates an isolated network that only my VMs can access, which is good for security testing later.
Network Settings:

Network Mode: Host-Only Adapter (vboxnet0)
Ubuntu Desktop (Workstation) IP: 192.168.56.10
Ubuntu Server IP: 192.168.56.11
Subnet Mask: 255.255.255.0

Both VMs can see each other on this network, and I can SSH from the workstation to the server using the server's IP address. This setup is isolated from my main network, so any security testing I 
do won't affect other devices.

System Specifications Using CLI
Here are the system specifications from my Ubuntu Server:
Command: uname -a

This shows my kernel version and that I'm running a 64-bit system.

Command: free -h

I allocated 2GB of RAM to my server, and currently only using about 450MB.

Command: df -h

My server has a 25GB virtual hard drive with about 20GB still free.

Command: ip addr

This confirms my server IP address is 192.168.56.11 on the host-only network.

Command: lsb_release -a

This confirms I'm running Ubuntu Server 22.04 LTS 
Week 1 Reflection:
Setting up the VMs was straightforward. I had to make sure both VMs were on the same host-only network so they could communicate. The server installed without a GUI as expected, which felt 
strange at first, but I can access it from my Ubuntu Desktop VM terminal. In phase 2, I'll start configuring SSH and security settings.
