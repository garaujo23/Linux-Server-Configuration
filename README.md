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

## Setup Web Application

1. Install Apache2 and required mod-wsgi:

```command
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi python-dev
```

2. Start mod_wsgi: `sudo a2emod wsgi`
3. Start the web server: `sudo service apache2 start`
4. Check this is working by visiting the public IP of your server to see the default page of Apache2. This will show that your firewall rules are working as well on port 80 (http).
5. Create a folder for your web app: `cd /var/www` , `sudo mkdir catalog` , `sudo chown -R grader:grader catalog` , `cd catalog`
6. Clone your web application: `sudo apt-get install git` , `git clone <your repo link>`
7. Create a wsgi file: `sudo vim /var/www/catalog/catalog.wsgi` and include:

```wsgi
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'your_super_secret_key'
```
8. Rename your `main.py` to `__init__.py` : `mv main.py __init__.py`
9. Create a virtual environment to install dependencies:

```command
sudo apt-get install python-pip
sudo pip install virtualenv
sudo virtualenv venv
source venv/bin/activate
sudo chmod -R 777 venv
```

10. Install dependencies: `sudo pip install Flask httplib2 oaythclient sqlalchemy psycopg2 requests`
11. Create, configure and enable the virtual hosts: ` sudo vim /etc/apache2/sites-available/catalog.conf`
12. Use the following:

```code
<VirtualHost *:80>
    ServerName <YOUR PUBLIC IP ADDRESS>
    ServerAlias <YOUR HOST NAME>
    ServerAdmin admin@<YOUR PUBLIC IP ADDRESS>
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

## Create Database & Deploy

1. Install required packages:

```command
sudo apt-get install libpq-dev python-dev
sudo apt-get install postgresql postgresql-contrib
sudo -u postgres -i
```

2. Enter psql command line: `psql`
3. Create a database, user and set permissions:

```command
CREATE USER catalog WITH PASSWORD <your password>;
ALTER USER catalog CREATEDB;
CREATE DATABASE catalog WITH OWNER catalog;
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
```

4. Change all reference to DB in your web app to `engine = create_engine('postgresql://catalog:<your_password>@localhost/catalog)`
5. Restart Apache server to launch your full stack web application! `sudo service apache2 restart`

## Reference
* https://github.com/chuanqin3/udacity-linux-configuration
* https://github.com/twhetzel/ud299-nd-linux-server-configuration
* https://github.com/boisalai/udacity-linux-server-configuration#step_5_1
* https://github.com/juvers/Linux-Configuration
* [Fail2Ban](http://www.fail2ban.org/wiki/index.php/Main_Page)
* https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04
* https://www.digitalocean.com/community/tutorials/how-to-configure-the-apache-web-server-on-an-ubuntu-or-debian-vps
* [Postgresql](https://www.postgresql.org/docs/8.0/sql-createuser.html)