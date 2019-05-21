# Linux-Server-Configuration

Udacity Full Stack web development nanodegree assignment 3 to take a baseline installation of a Linux distribution on a virtual machine and prepare it to host a web application, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers. Webserver stack is:
1. Linux 16.04
2. Apache2
3. PostgreSQL

## Server Details

Host name: <http://104.197.242.30.xip.io/>

Public IP: 104.197.242.30

## Google Cloud Server Setup

1. Go to [cloud console](https://console.cloud.google.com/)
2. Click Compute Engine and VM instances.
3. Create new instance. Free is a micro service but regardless include Ubuntu 16.04 and 50Gb SSD drive.
4. Once created, click SSH connect to login to server.
5. Click three dots and go to network settings. Add any firewall rules needed such as http(80), ssh(2200) and ftp(123)

## Server Setup

1. Update and upgrade installed packages

```command
sudo apt-get update
sudo apt-get upgrade
```

2. Change the SSH default port from 22 to 2200 by editing /etc/ssh/sshd_config: `sudo vim /etc/ssh/sshd_config`
3. Restart the SSH service: `sudo service ssh restart`
4. Configure the default Ubuntu firewall to only allow incoming connection on HTTP(80), SSH(2200) and NTP (123).

```command
sudo ufw status                  # Should be inactive.
sudo ufw default deny incoming   # Deny any incoming traffic.
sudo ufw default allow outgoing  # Enable outgoing traffic.
sudo ufw allow 2200/tcp          # Allow incoming tcp packets on port 2200 for SSH.
sudo ufw allow www               # Allow HTTP traffic.
sudo ufw allow 123/udp           # Allow incoming udp packets on port 123.
sudo ufw deny 22                 # Deny tcp and udp packets on port 22.
```

1. Enable UFW with `sudo ufw enable`
2. Check the status and all current rules with `sudo ufw status`
3. Install [Fail2ban](http://www.fail2ban.org/wiki/index.php/Main_Page) `sudo apt-get install fail2ban`
4. To send email `sudo apt-get install sendmail iptables-persistent`
5. Copy jail.conf to local copy `sudo cp/etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
6. Edit jail.local to change ssh to port 2200 and configure action send email and recipient. Restart the service `sudo service fail2ban restart`

```
destemail = user@domain
action = %(action_mwl)s
port = 2200 
```

## Create User Grader

1. To create the grader user: `sudo adduser grader`
2. To give permissions: `sudo visudo`
3. Add below root ALL=... `grader ALL=(ALL:ALL)`
4. To verify run: `su - grader` and enter password then run: `sudo - grader`. Make sure the last line includes your ALL statement from above.  
5. On your local machine, create an SSH key pair for user grader. This can be done using `ssh-keygen` or [Putty Gen](https://www.ssh.com/ssh/putty/windows/puttygen) for Windows.
6. On the virtual machine create an ssh directory: `mkdir .ssh`
7. Create an authorized keys file: `sudo vim ~/.ssh/authorized_keys`
8. In this file copy the public key you generated on your local machine into this file and save.
9. Give permissions: `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`
10. Change to deny password auth: `vim /etc/ssh/sshd_config` set `PasswordAuthetication no`
11. Restart service: `sudo service ssh restart`
12. On your local machine test by connecting with Putty on Windows or `ssh -i <path to private key> -p 2200 grader@<public ip>`