# Project 5

## Running

Running catalog app can be found at: http://ec2-52-24-229-226.us-west-2.compute.amazonaws.com/genres/ (to be able to log in)

To ssh to the box, run `ssh -i ~/.ssh/udacity_key.rsa grader@52.24.229.226 -p 2200`

## Solution description

In order to complete the project the following steps were taken:

### 1. Launch your Virtual Machine with your Udacity account.

I have created a virtual environment according to the instructions and was given IP: 52.24.229.226.

### 2. Follow the instructions provided to SSH into your server

I have successfully SSHed into the server.

### 3. Create a new user named grader

I've created a new user called grader using command `sudo adduser grader` and set a secure password for it.

### 4. Give the grader the permission to sudo

I've given the grader user permission to sudo using command `sudo usermod -a -G sudo grader` (http://askubuntu.com/questions/168280/how-do-i-grant-sudo-privileges-to-an-existing-user)

### 5. Update all currently installed packages

I've updated all installed packages using `sudo apt-get update` and `sudo apt-get upgrade`

### 6. Change the SSH port from 22 to 2200

I changed the SSH port to 2200 in `/etc/ssh/sshd_config` followed by `sudo restart ssh` (http://www.liquidweb.com/kb/changing-the-ssh-port/)

### 7. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

I've configured the firewall to only allow specified connections (https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server)

`sudo ufw allow 2200/tcp`

`sudo ufw allow 80/tcp`

`sudo ufw allow ntp`

`sudo ufw enable`

### 8. Configure the local timezone to UTC

I configured the local timezone to UTC (https://www.digitalocean.com/community/tutorials/additional-recommended-steps-for-new-ubuntu-14-04-servers)

`sudo dpkg-reconfigure tzdata`

`sudo apt-get install ntp`

### 9. Install and configure Apache to serve a Python `mod_wsgi` application

I installed and configured Apache (https://www.digitalocean.com/community/tutorials/installing-mod_wsgi-on-ubuntu-12-04):

`sudo apt-get install apache2`

`sudo apt-get install libapache2-mod-wsgi`

### 10. Install and configure PostgreSQL:

    * Do not allow remote connections
    * Create a new user named catalog that has limited permissions to your catalog application database

I have installed PostgreSQL:

`sudo apt-get install postgresql postgresql-contrib`

I configured it to not allow remote connections

`sudo vi /etc/postgresql/9.3/main/pg_hba.conf` and verified that remote connections are not possible (default option)

I created a new user named catalog with limited permissions to catalog application database:

`sudo -i -u postgres`

`psql`

`postgres=# CREATE USER catalog;`

`postgres=# ALTER USER catalog PASSWORD 'catalog';`

`postgres=# ALTER ROLE catalog NOSUPERUSER;`

`postgres=# CREATE DATABASE catalog WITH USER catalog;`

### 11. Install git, clone and setup your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser. Remember to set this up appropriately so that your .git directory is not publicly accessible via a browser!

I have installed git and created a key to enable at github (two-factor auth):

`sudo apt-get install git`

`ssh-keygen`

I cloned my Catalog App project:

`git clone git@github.com:aszwemin/fullstack-nanodegree-project3-solo.git`

I set up Apache to serve my app (https://www.digitalocean.com/community/tutorials/using-mod_wsgi-to-serve-applications-on-ubuntu-12-04):

* I created wsgi file `/var/www/catalog/catalog.wsgi`

```
#!/usr/bin/python

import sys
import logging

logging.basicConfig(stream=sys.stderr)

sys.path.insert(0,"/var/www/catalog")

from catalog.project import app as application
```

* And configured and enabled virtual host `/etc/apache2/sites-available/catalog.conf`

```
<VirtualHost *:80>
        ServerName 52.24.229.226

        WSGIScriptAlias / /var/www/catalog/catalog.wsgi

        <Directory /var/www/catalog>    
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

`sudo a2ensite catalog`

* I set up the permissions correctly:

`sudo find /var/www -type d -exec chmod 755 {} \;`

`sudo find /var/www -type f -exec chmod 644 {} \;`

`sudo chmod o+rx /var/www/catalog/`

`sudo chmod o+r /var/www/catalog/catalog.wsgi`

I set up my app to access postgres and setup the database and prepopulated data:

`apt-get install libpq-dev`

`apt-get install python-dev`

`pip install psycopg2`

`python database_setup.py`

`python prepopulate.py`

I've installed all the libraries required by the app.

I made login options run by getting the hostname through http://www.hcidata.info/host2ip.cgi and configuring Google/Facebook
