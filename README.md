### devops-playground

# VM Part
# You have to run a Virtual Machine (VM) with the Linux OS of your choice (Debian
# Jessie, CentOS 7...) in the hypervisor of your choice (VMWare Fusion, VirtualBox...).
# • A disk size of 8 GB.
# • Have at least one 4.2 GB partition.
# • It will also have to be up to date as well as the whole packages installed to meet
# the demands of this subject.

------

- Install Virtual Box
- Download Debian from https://www.debian.org/distrib/. I choosed amd-64 small iso distrib
- Install Debian to your Virtual Box.
--- Choose 8GB size, VDI
--- Manual Instalation worked for me
--- Choose manual Partition method.
--- Create one 4.2GB Ext4 for '/'
--- Probably create 3GB Ext4 for '/home'
--- The rest - swap area
--- Don't install any GUI
--- Install GRUB as prompted

# You must create a non-root user to connect to the machine and work.
- You've created it during the installation
If you'd like to create another user 
```
adduser [username]
id [username]
```
