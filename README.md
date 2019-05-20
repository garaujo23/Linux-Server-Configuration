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
3. 