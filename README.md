# devops-playground

### VM Part
### You have to run a Virtual Machine (VM) with the Linux OS of your choice (Debian
### Jessie, CentOS 7...) in the hypervisor of your choice (VMWare Fusion, VirtualBox...).
### • A disk size of 8 GB.
### • Have at least one 4.2 GB partition.
### • It will also have to be up to date as well as the whole packages installed to meet
### the demands of this subject.

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
##
### You must create a non-root user to connect to the machine and work.
- You've created it during the installation
If you'd like to create another user 
```
adduser [username]
id [username]
```
### Use sudo, with this user, to be able to perform operation requiring special rights.
- Enter root mode
```
su
apt-get update -y && apt-get upgrade -y
apt-get install sudo 
exit
```
- Add user to the sudoers file
```
$ pwd
cd /etc/
chmod +w sudoers
nano sudoers
```
- Add a line "[username] ALL=(ALL:ALL)"

```
exit
```

### We don’t want you to use the DHCP service of your machine. You’ve got to configure it to have a static IP and a Netmask in \30.

https://www.codesandnotes.be/2018/10/16/network-of-virtualbox-instances-with-static-ip-addresses-and-internet-access/

- Check the article. Create a Host Network Manager instance. Use default ip address but the mask must be changed to 255.255.255.252. Disable DHCP tab
- Power off your Debian. In it's setting (network tab) enable the second adapter and attach it to Host Only Adapter
- In your Debian - create two files in /etc/network/interfaces.d
* enp0s3 // internate access
```
iface enp0s3 inet dhcp
```
* enp0s8 // Host Only network
```
iface enp0s8 inet static
  address [Check your Host Only Adapter Address - if it's 192.168.56.101 then enter 192.168.56.102 (Our netmask let us only 2 addresses to have)]
  netmask 255.255.255.252
```
- update /etc/network/iterfaces file like that: 
```
...
#the primary network interfaces
auto enp0s3
auto enp0s8
``` so that the interfaces reads from the separate files from interfaces_d/ folder

```
sudo service networking restart
```
- check with sudo ifconfig
