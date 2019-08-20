# Udacity Linux Server Configuration

Udacity Full Stack Web Developer Nanodegree Project

## About

In this project, a web application is served from a remote linux server.

- The IP address of the server: 172.105.91.153
- SSH Port: 2200
- The URL to web application: http://172.105.91.153

## Steps
##### Create Linode Account and a Virtual Linux Server
- Create a Linode account: https://www.linode.com/
- Create an Ubuntu Server and connect to it: https://www.linode.com/docs/getting-started/#create-a-linode

##### Create User
- Create user grader: `sudo adduser grader`
- Give grader sudo access:
    + Create sudoers file for grader and open: `sudo nano /etc/sudoers.d/grader`
    + Add following line to file and save it:  `grader ALL=(ALL:ALL) ALL`

##### Configurate SSH
- Generate key pairs on your local machine: `ssh-keygen`
- Place the public key on remote server:
    + Create directory ./ssh within home directory: `mkdir .ssh`
    + Create file for public keys:  `touch .ssh/authorized_keys`
    + Copy the contents of the .pub file in your local machine and paste to .ssh/authorized keys file on remote server: `nano .ssh/authorized_keys`
- Change permissons of .ssh directory:
    + `chmod 700 .ssh`
    + `chmod 644 .ssh/authorized_keys`
- Disable password login:
    + Open SSH configuration file: `sudo nano /etc/ssh/sshd_config`
    + Change line `PasswordAuthentication yes` to  `PasswordAuthentication no`
- Disable root login:
    + In SHH configuration file, change line `PermitRootLogin yes` to  `PermitRootLogin no`
- Change SSH port:
    + In SHH configuration file, change line `Port 22` to  `Port 2200`
- Restart SSH service:
`sudo service ssh restart`

##### Update all system packages to most recent versions
- Update packages: `sudo apt update && sudo apt dist-upgrade`
- Reboot the system: `sudo reboot`

##### Set Up Firewall
- Deny all incoming requests by default: `sudo ufw default deny incoming`
- Allow all outgoing requests by default: `sudo ufw default allow outgoing`
- Allow HTTP requests: `sudo ufw allow 80/tcp`
- Allow SSH requests: `sudo ufw allow 2200/tcp`
- Allow NTP requests: `sudo ufw allow 123/udp`
- Enable firewall `sudo ufw enable`

##### Clone Item Catalog Project
- Install git: `sudo apt-get install git`
- Create and go to directory to clone project:
    + `sudo mkdir /var/www/UdacityItemCatalog`
    + `cd /var/www/UdacityItemCatalog`
- Clone item catalog project: `sudo git clone https://github.com/MerveNurYilmaz/Udacity-Item-Catalog.git`
- Change name of the project folder: `sudo mv ./Udacity-Item-Catalog ./UdacityItemCatalog`
- Change name of the application.py to __init__.py: `sudo mv application.py __init__.py`

##### Create Database
- Install postgresql: `sudo apt-get install postgresql`
- Create a user and database
- Change all engine variables in the item catalog project from
`engine = create_engine("sqlite:///shoppingcatalog.db")` to
`engine = create_engine('postgresql://DBUSER:USERPASSWORD@localhost/DBNAME')`
- Populate DB: `python /var/www/UdacityItemCatalog/UdacityItemCatalog/populate_db.py`

##### Configure Apache Server
- Install apache and mod_wsgi:
    + `sudo apt-get install apache2`
    + `sudo apt-get install python-setuptools libapache2-mod-wsgi`
- Create wsgi:
    + Create wsgi file: `sudo nano /var/www/UdacityItemCatalog/itemcatalog.wsgi`
    + Paste into wsgi file:
```
	#!/usr/bin/python
	import sys
	import logging
	import flask
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/UdacityItemCatalog/")
	print(sys.path)

	from UdacityItemCatalog import app as application
	application.secret_key = 'supersecretkey'
```
- Create apache configuration file:
    + Create apache configuration file: `sudo nano /etc/apache2/sites-available/UdacityItemCatalog.conf`
    + Paste into conf file:
```
	<VirtualHost *:80>
		ServerName 172.105.91.153
		WSGIScriptAlias / /var/www/UdacityItemCatalog/itemcatalog.wsgi
		<Directory /var/www/UdacityItemCatalog/UdacityItemCatalog/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/UdacityItemCatalog/UdacityItemCatalog/static
		<Directory /var/www/UdacityItemCatalog/UdacityItemCatalog/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
```
- Enable virtaul host: `sudo a2ensite FlaskApp`
- Restart Apache: `sudo service apache2 restart`



## References
https://www.linode.com/docs/getting-started/#create-a-linode
https://www.udacity.com/course/configuring-linux-web-servers--ud299
https://www.learnlinux.tv
