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
--- Check ssh services for installation
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
```
so that the interfaces reads from the separate files from interfaces_d/ folder

```
sudo service networking restart
```
- check with
```
sudo ifconfig
```
### You have to change the default port of the SSH service by the one of your choice. SSH access HAS TO be done with publickeys. SSH root access SHOULD NOT be allowed directly, but with a user who can be root.

- Edit /etc/ssh/sshd_config . 
- Uncomment #Port 22 and change it to whatever port you like

```
sudo service sshd restart
```
- You could try ssh on your Host computer
```
sudo ssh [username]@[ip address of your enp0s8 on server] -p [the port you've entered to sshd_config]
```
- Generate ssh on your Host computer
```
ssh-keygen -t rsa
```
- check this to add the ssh public keys to your Debian machine in order to let the Host connect without a password https://www.youtube.com/watch?v=hQWRp-FdTpc
- when you've done with the keys - disable root and password connections in /sshd_config
```
PermitRootLogin no
PasswordAuthentication no
```
### You have to set the rules of your firewall on your server only with the services used outside the VM.
https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-debian-9
```
sudo apt-get install ufw
sudo ufw enable
sudo ufw allow [your ssh port]/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443
```
### You have to set a DOS (Denial Of Service Attack) protection on your open ports of your VM.

```
sudo apt-get install iptables fail2ban apache2
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```
- Add those rules to jail.local to protect all port 80 attack. SSH is protected by default
*
[http-get-dos]
enabled = true
port = http,https
filter = http-get-dos
logpath = %(apache_error_log)s
maxretry = 300
findtime = 300
#ban for 5 minutes
bantime = 600
action = iptables[name=HTTP, port=http, protocol=tcp]
*

- Add the following file
```
cat /etc/fail2ban/filter.d/http-get-dos.conf
# Fail2Ban configuration file
#
# Author: http://www.go2linux.org
#
[Definition]

# Option: failregex
# Note: This regex will match any GET entry in your logs, so basically all valid and not valid entries are a match.
# You should set up in the jail.conf file, the maxretry and findtime carefully in order to avoid false positives.

failregex = ^ -.*GET

# Option: ignoreregex
# Notes.: regex to ignore. If this regex matches, the line is ignored.
# Values: TEXT
#
ignoreregex =
```

```
 sudo ufw reload
 sudo service fail2ban restart
 // check 
 sudo fail2ban-client status
```

### You have to set a protection against scans on your VM’s open ports. 
```
sudo apt-get install portsentry
```
- in /etc/default/portsentry:
```
TCP_MODE="atcp"
UDP_MODE="audp"
```

- in /etc/portsentry/portsentry.conf
```
BLOCK_UDP="1"
BLOCK_TCP="1"
...
comment - KILL_ROUTE="/sbin/route add -host $TARGET$ reject"
uncomment - KILL_ROUTE="/sbin/iptables -I INPUT -s $TARGET$ -j DROP"
```

```
sudo /etc/init.d/portsentry start
```
logs >>  /var/log/syslog file.

### Stop the services you don’t need for this project.
- find
```
 ls /etc/init.d
```
- disable
```
sudo systemctl disable [servicename].service
```

### Create a script that updates all the sources of package, then your packages and which logs the whole in a file named /var/log/update_script.log. Create a scheduled task for this script once a week at 4AM and every time the machine reboots.

```
touch [name].sh
chmod 777 [name].sh
echo "sudo apt-get update -y >> /var/log/update_script.log" >> [name].sh 
echo "sudo apt-get upgrade -y >> /var/log/update_script.log" >> [name].sh 

sudo crontab -e

>>> 
0 4 * * MON /home/[user]/[anything].sh

```

###Make a script to monitor changes of the /etc/crontab file and sends an email to root if it has been modified. Create a scheduled script task every day at midnight.
```
  touch monitor.sh
  chmod 777 monitor.sh
```
 
 ```
 root@ghostsys:/home/ghost# cat monitor.sh 
sudo touch /home/ghost/cron_md5
sudo chmod 777 /home/ghost/cron_md5
m1="$(md5sum '/etc/crontab' | awk '{print $1}')"
m2="$(cat '/home/ghost/cron_md5')"

if [ "$m1" != "$m2" ] ; then
	md5sum /etc/crontab | awk '{print $1}' > /home/ghost/cron_md5
	echo "KO" | mail -s "Cronfile was changed" root@debian.lan
fi
```

```
sudo crontab -e
>>>
* * * * * /home/ghost/monitor.sh
```

### Mails
```
sudo apt install bsd-mailx
sudo apt install postfix
>> Local only
>> debian.lan

```
```
sudo nano /etc/aliases 
>>> 
...
root:root
```

```
sudo newaliases
sudo postconf -e "home_mailbox = mail/"
sudo service postfix restart
sudo apt install mutt
sudo nano /root/.muttrc
>>
set mbox_type=Maildir
set folder="/root/mail"
set mask="!^\\.[^.]"
set mbox="/root/mail"
set record="+.Sent"
set postponed="+.Drafts"
set spoolfile="/root/mail"
```

```
  mutt
  q
  echo "Text" | sudo mail -s "Subject" root@debian.lan
```
