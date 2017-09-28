# Linux-Server-Configuration-UDACITY
This is the fifth project for "Full Stack Web Developer Nanodegree" on Udacity. 

In this project, a Linux virtual machine needs to be configurated to support the My Restaurant website.

You can visit my website here: http://13.59.131.173/ 

AWS- Server : ec2-13-59-131-173.us-east-2.compute.amazonaws.com

## Tasks
1. Launch your Virtual Machine with your Udacity account
2. Follow the instructions provided to SSH into your server
3. Create a new user named grader
4. Give the grader the permission to sudo
5. Update all currently installed packages
6. Change the SSH port from 22 to 2200
7. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
8. Configure the local timezone to UTC
9. Install and configure Apache to serve a Python mod_wsgi application
10. Install and configure PostgreSQL:
	- Do not allow remote connections
	- Create a new user named catalog that has limited permissions to your catalog application database
11. Install git, clone and setup your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser. Remember to set this up appropriately so that your .git directory is not publicly accessible via a browser!

## Launch Virtual Machine

## Instructions for SSH access to the instance
1. Download Private Key below
2. Move the private key file into the folder `~/.ssh` (where ~ is your environment's home directory). So if you downloaded the file to the Downloads folder, just execute the following command in your terminal.
	```mv ~/Downloads/udacity_key.rsa ~/.ssh/```
3. Open your terminal and type in
	```chmod 600 ~/.ssh/udacity_key.rsa```
4. In your terminal, type in
	```ssh -i ~/.ssh/udacity_key.rsa root@13.59.131.173```
5. Development Environment Information

	Public IP Address

	13.59.131.173
	
	Private Key ( is not provided here. )

## Create a new user named grader
1. `sudo adduser grader`
2. `nano /etc/sudoers`
3. `touch /etc/sudoers.d/grader`
4. `nano /etc/sudoers.d/grader`, type in `grader ALL=(ALL:ALL) ALL`, save and quit

## Set ssh login using keys
1. generate keys on local machine using`ssh-keygen` ; then save the private key in `~/.ssh` on local machine
2. deploy public key on developement enviroment

	On you virtual machine:
	```
	$ su - grader
	$ mkdir .ssh
	$ touch .ssh/authorized_keys
	$ nano .ssh/authorized_keys
	```
	Copy the public key generated on your local machine to this file and save
	```
	$ chmod 700 .ssh
	$ chmod 644 .ssh/authorized_keys
	```
	
3. reload SSH using `service ssh restart`
4. now you can use ssh to login with the new user you created

	`ssh -i [privateKeyFilename] grader@13.59.131.173`

## Update all currently installed packages

	sudo apt-get update
	sudo apt-get upgrade

## Change the SSH port from 22 to 2200
1. Use `sudo nano /etc/ssh/sshd_config` and then change Port 22 to Port 2200 , save & quit.
2. Reload SSH using `sudo service ssh restart`

## Configure the Uncomplicated Firewall (UFW)

Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
	 
	sudo ufw status
	sudo ufw allow 2200/tcp
	sudo ufw allow 80/tcp
	sudo ufw allow 123/udp
	sudo ufw enable

Now, check ufw status:
      
      sudo ufw status  
 
## Configure the local timezone to UTC
1. Configure the time zone `sudo dpkg-reconfigure tzdata`
2. It is already set to UTC.

## Install and configure Apache to serve a Python mod_wsgi application
1. Install Apache `sudo apt-get install apache2`
2. Install mod_wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`
3. Restart Apache `sudo service apache2 restart`

## Install and configure PostgreSQL
1. Install PostgreSQL `sudo apt-get install postgresql`
2. Check if no remote connections are allowed `sudo nano /etc/postgresql/9.3/main/pg_hba.conf`
3. Login as user "postgres" `sudo su - postgres`
4. Get into postgreSQL shell `psql`
5. Create a new database named catalog  and create a new user named catalog in postgreSQL shell
	
	```
	postgres=# CREATE DATABASE catalog;
	postgres=# CREATE USER catalog;
	```
5. Set a password for user catalog
	
	```
	postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
	```
6. Give user "catalog" permission to "catalog" application database
	
	```
	postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
	```
7. Quit postgreSQL `postgres=# \q`
8. Exit from user "postgres" 
	
	```
	exit
	```
 
## Install git, clone and setup your Catalog App project.
1. Install Git using `sudo apt-get install git`
2. Use `cd /var/www` to move to the /var/www directory 
3. Create the application directory `sudo mkdir FlaskApp`
4. Move inside this directory using `cd FlaskApp`
5. Clone the Catalog App to the virtual machine `git clone https://github.com/apexa11/Restaurant_Menu-App.git`
6. Rename the project's name `sudo mv ./Restaurant_Menu-App ./FlaskApp`
7. Move to the inner FlaskApp directory using `cd FlaskApp`
8. Rename `project.py` to `__init__.py` using `sudo mv project.py __init__.py`
9. Edit `database_setup.py`, `project.py` and `lotsofmenus.py` and change `engine = create_engine('sqlite:///restaurantmenuwithusers.db')` to `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
10. Install pip `sudo apt-get install python-pip`
11. sudo apt-get install python-setuptools
12. sudo pip install Flask
13. pip install httplib2
14. pip install requests
15. sudo pip install flask-seasurf
16. sudo apt-get install python-psycopg2
17. sudo pip install oauth2client
18. sudo pip install sqlalchemy
19. Use pip to install dependencies `sudo pip install -r requirements.txt`
20. Install psycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`
21. Create database schema `sudo python database_setup.py`

Now, you can check your database connection by following commands:
1. sudo su - postgres
2. psql
3. \c catalog
4. \dt

## Configure and Enable a New Virtual Host
1. Create FlaskApp.conf to edit: `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
2. Add the following lines of code to the file to configure the virtual host. 
	
	```
	<VirtualHost *:80>
		ServerName 13.59.131.173
		ServerAdmin admin@13.59.131.173
		WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
		<Directory /var/www/FlaskApp/FlaskApp/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/FlaskApp/FlaskApp/static
		<Directory /var/www/FlaskApp/FlaskApp/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
	```
3. Enable the virtual host with the following command: `sudo a2ensite FlaskApp`

## Create the .wsgi File
1. Create the .wsgi File under /var/www/FlaskApp: 
	
	```
	cd /var/www/FlaskApp
	sudo nano flaskapp.wsgi 
	```
2. Add the following lines of code to the flaskapp.wsgi file:
	
	```
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/FlaskApp/")

	from FlaskApp import app as application
	application.secret_key = 'Add your secret key'
	```

## Restart Apache
1. Restart Apache `sudo service apache2 restart `

## If you are getting Internal server error, You can access the Apache error log file. To view the last 30 lines in the error log,you have to use following command
 sudo tail -30 /var/log/apache2/error.log

## K - Get OAUTH-LOGINS (Google+ and Facebook) working.

1.To fix the google: g_client_secrets.json error, go to the login session of your application, to these sections:

app_token = json.loads(
open('g_client_secret.json', 'r').read())['web']['client_id']

oauth_flow:flow_from_clientsecrets('g_client_secret.json', scope='')
Add /var/www/Catalog/catalog to your code path.:

app_token = json.loads(
open(r'/var/www/Catalog/catalog/g_client_secret.json', 'r').read())['web']['client_id']


oauth_flow:flow_from_clientsecrets('/var/www/FLaskApp/FlaskApp/g_client_secret.json', scope='')
Do the same thing for the fb_client_secret.json file. This is to enable apache locate the file through the proper path.

2. If you are Go to http://www.hcidata.info/host2ip.cgi to recieve the Host Name for your PUBLIC-IP-ADDRESS. The Host Name for mine : 13.59.131.173 is ec2-13-59-131.173.us-west-2.compute.amazonaws.com

3. Open Apache catalog.conf file.
Ugrader@ip-10-20-30-101:/var/www/FlaskApp/FlaskApp$ sudo nano /etc/apache2/sites-available/FlaskApp.conf
Paste this in the nano editor for catalog.conf: ServerAlias ec2-13-59-131.173.us-west-2.compute.amazonaws.com

<VirtualHost *:80>
    ServerName 13.59.131.173
    ServerAdmin admin@13.59.131.173
    ServerAlias ec2-13-59-131.173.us-west-2.compute.amazonaws.com
    WSGIScriptAlias / /var/www/FlaskApp/FlaskApp.wsgi
    <Directory /var/www/FlaskApp/FlaskApp/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/FlaskApp/FlaskApp/static
    <Directory /var/www/FlaskApp/FlaskApp/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
Enable virtual host - catalog.conf

4. grader@ip-10-20-30-101:/var/www/FlaskApp/FlaskApp$ sudo a2ensite catalog
To get Google+ authorization working, do this:

(a) - On the Developer Console: http://console.developers.google.com, select your Project.

(b) - Navigate to Credentials, and edit your OAuth 2.0 client IDs like so:

#oauth2

5. To get Facebook authorization working, do this:

(a) - Go to Facebook developers page: `https://developers.facebook.com/apps and select your app.

(b) - Go to settings and fill in your PUBLIC-IP-ADDRESS like so:

# facebook oauth

6. You can install Monitor application:

 grader@ip-10-20-30-101:/var/www/FLaskApp/FlaskApp$ sudo apt-get install python-pip build-essential python-dev
 grader@ip-10-20-30-101:/var/www/FlaskApp/FlaskApp$ sudo pip install Glancers
L - Install and Configured Fail2ban intrusion protection that bans suspicious IPs.

#Reference: DigitalOcean

7. Install Fail2ban application:
grader@ip-10-20-30-101:~$ sudo apt-get install fail2ban

8. Copy the default config file
grader@ip-10-20-30-101:~$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

9. Open jail.local and change the followinf default parameters:
grader@ip-10-20-30-101:~$ sudo nano /etc/fail2ban/jail.local

10. Set the following parameters:
set bantime = 1600
destemail = YOURNAME@DOMAIN or YOUR-EMAIL
action = %(action_mwl)s
under [ssh] change port = 2200

11. Stop the service:
grader@ip-10-20-30-101:~$ sudo service fail2ban stop
Start the service again:

12. grader@ip-10-20-30-101:~$ sudo service fail2ban start

13. FINALLY:
Restart apache2 server, run your app on amazonaws.

## References:
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
