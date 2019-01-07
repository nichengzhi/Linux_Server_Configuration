# Linux_Server_Configuration

### prject summary
implement a apache2 server on AWS lightsail ubutunu instance.

- IP Address: http://100.27.1.69/ 
- SSH PORT: 2200

### step

1. update packages in linux server and disable root login:
	- sudo apt-get update
	- sudo apt-get upgrade
	- sudo apt-get dist-upgrade
	- sudo vim /etc/ssh/sshd_config
	- change PermitRootLogin withou-pasword to no
2. create a user called grader:
	- sudo adduser grader
	- sudo vim /etc/sudoers.d/grader
	- add "grader ALL=(ALL:ALL) ALL" to this file
3. change port from 22 to 2200:
	- sudo vim /etc/ssh/sshd_config
	- change 22 to 2200
4. set firewall confirguation
	- sudo ufw allow 2200/tcp (you also need set tcp port 2200 in amazon AWS page)
	- sudo ufw allow 80/tcp
	- sudo ufw allow 123/udp
	- sudo ufw allow www
	- sudo ufw enable
5. change timezone to utc
	- sudo dpkg-reconfigure tzdata then select utc
6. public key encryption
	- in local machine: ssh-keygen and set keyfile name
	- in lightsail server: 
	- su - grader
	- mkdir .ssh
	- touch .ssh/authorized_keys
	- nano .ssh/authorized_keys 
	- chmod 700 .ssh
	- chmod 644 .ssh/authorized_keys
	- paste local key.pub information to authorized_keys file
	- then you can use:ssh grader@100.27.1.69 -p 22 -i ~/.ssh/lightsailgrader to login to the lightsail server

7.  web server confirguation
	- install apache: sudo apt-get install apache2
	- install mode_wsgi: sudo apt-get install libapache2-mod-wsgi python-dev
	- clone my previouse web app project: 
	- cd /var/www -> sudo mkdir catalog -> cd catalog -> git clone https://github.com/nichengzhi/udacity_item_catalog.git
	- change project.py to __init__.py: mv project.py __init__.py
	- sudo vim catalog.wsgit add the script below:
	'''
	  import sys
	  import logging
	  logging.basicConfig(stream=sys.stderr)
	  sys.path.insert(0, "/var/www/catalog/")
	  
	  from udacity_item_catalog import app as application
	  application.secret_key = 'supersecretkey'
	'''
	- install virtual env for better package management
	- sudo pip install virtualenv
	- sudo virtualenv project
	- source project/bin/activate
	- sudo chmod -R 777 project

	- install pip: sudo apt-get install python-pip
	- install project packages: sudo pip install sqlalchemy psycopg2 ...
	- change file path in __init__.py 
	- create a new virtual host:
	- sudo vim /etc/apache2/sites-available/catalog.conf
	'''
	<VirtualHost *:80>
    ServerName 100.27.1.69
    ServerAlias ec2-100-27-1-69.us-west-2.compute.amazonaws.com
    ServerAdmin admin@100.27.1.69
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/project/lib/python2.7/site-packages
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/udacity_item_catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/udacity_item_catalog/static
    <Directory /var/www/catalog/udacity_item_catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
	'''
8. install postgrel
	- sudo apt-get install postgresql
	- sudo -u postgres psql
	- create user called catalog in psql command line and create a database called catalog
	- CREATE USER catalog WITH PASSWORD 'password';
	- CREATE DATABASE catalog WITH OWNER catalog;
	change the database path in database_setup.py additems.py __init__.py to postgresql://catalog:catalog@localhost/catalog
	- create table and ingest data by run database_setup.py and additems.py
	- to prevent database connected by remote:
	- sudo vim /etc/postgresql/9.5/main/pg_hba.conf
	- make sure the file has the content listed below:
	'''
	# Database administrative login by Unix domain socket
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5

	'''
9. reboot the server
	- use the virtual host set above: sudo a2ensite catalog
	- sudo service apache2 restart
	- visit http://100.27.1.69/

