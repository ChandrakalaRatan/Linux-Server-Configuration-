# Udacity Configuring a Linux server Project.
This project is a part of Udacity Full-Stack Developer Nanodegree. This project is to configure a Linux server to host a web app securely.

This document describes configuration of a baseline installation of a ubuntu Linux distribution and hosting a web application on it. The server is secured from a number of attack vectors, its software is updated, web and database servers are installed.

The hosted web application which is written in Python Flask framework and PostgreSQL is used as a database, and Apache2 with mod_wsgi is the webserver.

# Server details

 ## Public IP : 99.79.42.115

 ## SSH port : 2200

# Summary of software installed and configuration 

## Create Lightsail Server Instance

	1. Create a login account in Amazon Web Services.  
	2. Login into Lightsail. 
	3. To create virtual private server instance, 
	4. Choose Linux and OS Only option and select Ubuntu 16.04.LTS.
	5. Choose lowest price instance plan
	6. Give a unique host name to you instance.
	7. Type the following on Lanuch script textarea
		apt-get –y update  # update package list
		apt-get install –y nodejs  # install some of my favorite tools
	8. create a new SSH key pair as CatalogProject and choose location as Montreal, Zone A 
	9. This creates a new instance and with public IP and Private IP with Username to connect using SSH.
	10. Simply press Connect using SSH, it will start a new linux terminal(Lightsail Remote Server Terminal)


## On the Lightsail Remote Server Terminal Update all currently installed packages

	This is to ensure your system is secure is to keep your software up to date with new releases

	sudo apt update     				# update available package lists
	sudo apt upgrade    				# upgrade installed packages
	sudo apt autoremove 				# automatically remove packages that are no longer required


### Create a New User grader and give grader sudo

	sudo adduser grader 				# create a new user named grader # grader password is 'udacity'
	sudo passwd grader 				#run this command if require password change 
	sudo usermod -aG sudo grader 			#add grader to admine group that is(sudo) group.


### Set-up SSH keys for user grader
As root user
	sudo mkdir /home/grader/.ssh		       # create a directory called .ssh in grader 
	sudo chown grader:grader /home/grader/.ssh     # changing ownership of .ssh to grader
	sudo chmod 700 /home/grader/.ssh               # change folder permission
	
	sudo cp /home/ubuntu/.ssh/authorized_keys /home/grader/.ssh/
	sudo chown grader:grader /home/grader/.ssh/authorized_keys
	sudo chmod 644 /home/grader/.ssh/authorized_keys

### Create an SSH key pair for grader using the ssh-keygen tool at local  
	
	At local vagrant machine
		
		ssh-keygen  
		
	# Generating public/private rsa key pair	
        # Enter file in which to save the key (/Users/chandra/.ssh/id_rsa):  grader
	# empty passphrase
	# Verify at local vagrant machine a private key (grader) and a public key (grader.pub) are created. 
	
	
### Copying your Public Key Using SSH

	cat grader.pub | ssh -i grader grader@99.79.42.115 -p 2200 "cat >> ~/.ssh/authorized_keys" 
	
	# copy local grader.pub to authorized_keys file in remote server

    	ssh -i grader grader@99.79.42.115  # now we can loging without password

### Disable remote SSH login as root and Change the SSH port from 22 to 2200.

	sudo nano /etc/ssh/sshd_config                # change the following
		
	1. PermitRootLogin no                         # Disable remote SSH login as root
	2. PasswordAuthentication no                  # Disable password-based authentication for SSH
	3. Port 2200                                  # Change the SSH port from 22 to 2200

	sudo service ssh restart                      # restart ssh service
				
	ssh -i ~/.ssh/catalog_proj.rsa grader@99.79.42.115 -p 2200     # we can login with port 2200

### Configure the Uncomplicated Firewall (UFW) to only allow incoming 

	connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
	Warning: When changing the SSH port, make sure that the firewall is open for port 2200 first, so that you 
	don't lock yourself out of the server.
	When you change the SSH port, the Lightsail instance will no longer be accessible through the web app 'Connect 
	using SSH' button. The button assumes the default port is being used.

	sudo ufw status                 		# check ufw status 
	sudo ufw default deny incoming  		# block all incoming connections on all ports
	sudo ufw default allow outgoing 		# Allow outgoing connection on all ports:
	sudo ufw allow 2200/tcp         		# Allow incoming connection for SSH on port 2200
	sudo ufw allow www              		# Allow incoming connections for HTTP on port 80
	sudo ufw allow ntp              		# Allow incoming connection for NTP on port 123
	sudo ufw show added             		# Check the rules enabled before the firewall use
	sudo ufw enable                 		# Enable the firewall
	sudo ufw status                 		# Check the status of the firewall,

### Configure the local timezone to UTC.
	
	sudo timedatectl set-timezone UTC

### Install Apache and mod_wsgi for python3
	
	sudo apt install apache2                        # install apache
	sudo apt install libapache2-mod-wsgi-py3        # install python3 mod_wsgi

### Install and configure PostgreSQL
		
	sudo apt install postgresql                     # install postgreSQL
	sudo nano /etc/postgresql/9.5/main/pg_hba.conf  # only allowed connections from the local host 
							# addresses 127.0.0.1 for IPv4 and ::1 for IPv6.
									
	sudo -u postgres createuser -P catalog 		# Create a PostgreSQL user called catalog. You are prompted for a 
							# password. This creates a normal user that can't create databases, 
							# roles (users).

	sudo -u postgres createdb -O catalog catalog    # create an empty database called catalog

	sudo su - postgres                              #Login as user "postgres"

        psql                                            #Get into postgreSQL shell
	
	# Set a new password if required for user catalog
	
	postgres=# ALTER ROLE catalog WITH PASSWORD 'password';         
	
	# Give user "catalog" permission to "catalog" application database
	
        postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog; 
									
    	postgres=# \q  					#Quit postgreSQL 
    	postgres=# exit 			        #Exit from user "postgres"
		
### Install Flask, SQLAlchemy and dependencies
	
	sudo apt-get install python-psycopg2 python-flask
	sudo apt-get install python-sqlalchemy python-pip
	sudo apt-get install postgresql-plpython
	sudo pip install oauth2client
	sudo pip install requests
	sudo pip install httplib2
	sudo pip install flask-seasurf
	sudo pip install flask-httpauth

### Install Git version control software
        
	sudo apt-get install git

### Clone and setup Catalog project from the Github repository
	cd /var/www/
	sudo mkdir chandra_fullstack
	sudo chown grader:grader chandra_fullstack/
	sudo -u grader git clone https://github.com/ChandrakalaRatan/catalog.git catalog
	sudo chmod 700 /var/www/catalog/.git # Protect .git folder

### Update catalog.wsgi file for this installation
	Absolute paths are updated to where the catalog is located. The application secret_key is set to something random
	and the PostgreSQL for the catalog user is set. In this case, the file looks like this, except the password and 
	secret is not shown for security reasons.

	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0, '/var/www/chandra_fullstack/catalog')

	from catalog import app as application
	from catalog.database_setup import create_db
	from catalog.populate_database import populate_database

	application.secret_key = 'SECRET'  # This needs changing in production env

	application.config['DATABASE_URL'] = 'postgresql://catalog:PASSWORD@localhost/catalog'
	application.config['UPLOAD_FOLDER'] = '/var/www/chandra_fullstack/catalog/catalog/item_images'
	application.config['OAUTH_SECRETS_LOCATION'] = '/var/www/chandra_fullstack/catalog/'
	application.config['ALLOWED_EXTENSIONS'] = set(['jpg', 'jpeg', 'png', 'gif'])
	application.config['MAX_CONTENT_LENGTH'] = 4 * 1024 * 1024  # 4 MB

	# Create database and populate it, if not already done so.
	create_db(application.config['DATABASE_URL'])
	populate_database()


### Find the host name 
	Open http://www.hcidata.info/host2ip.cgi and receive the Host name for your public IP-address, e.g. for 
	99.79.42.115, its http://ec2-99-79-42-115.ca-central-1.compute.amazonaws.com

### Configure Apache2 to serve the app
	To serve the catalog app using the Apache web server, a virtual host configuration file needs to be created 
	in the directory /etc/apache2/sites-available/, in this case called catalog-app.conf. Here are its contents:

<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        
	ServerName http://99.79.42.115
	ServerAlias HOSTNAME, e.g. ec2-99-79-42-115.ca-central-1.compute.amazonaws.com
 	
	#ServerAdmin webmaster@localhost

        DocumentRoot /var/www/chandra_fullstack/catalog

        # Define WSGI parameters. The daemon process runs as the grader user.
        WSGIDaemonProcess catalog user=grader group=grader threads=5
        WSGIProcessGroup catalog
        WSGIApplicationGroup %{GLOBAL}

        # Define the location of the app's WSGI file
        WSGIScriptAlias / /var/www/chandra_fullstack/catalog/catalog.wsgi

        # Allow Apache to serve the WSGI app from the catalog app directory
        <Directory /var/www/chandra_fullstack/catalog/>
                Require all granted
        </Directory>

        # Setup the static directory (contains CSS, Javascript, etc.)
        Alias /static /var/www/chandra_fullstack/catalog/catalog/static

        # Allow Apache to serve the files from the static directory
        <Directory  /var/www/chandra_fullstack/catalog//catalog/static/>
                Require all granted
        </Directory>

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>


### Disable the default virtual host with:

	sudo a2dissite 000-default.conf

### Then enable the catalog app virtual host:

	sudo a2ensite catalog-app.conf

### To make these Apache2 configuration changes live, reload Apache:

	sudo service apache reload
	
### Update the Google OAuth client secrets file

	Fill in the client_id and client_secret fields in the file g_client_secrets.json. Also change the javascript_origins 
	field to the IP address and AWS assigned URL of the host. In this instance that would be: "javascript_origins":
	["http://99.79.42.115 ", "http://ec2-99-79-42-115.ca-central-1.compute.amazonaws.com"]

	These addresses also need to be entered into the Google Developers Console -> API Manager -> Credentials, in the web
	client under "Authorized JavaScript origins".

### Update the Facebook OAuth client secrets file
	
	In the file fb_client_secrets.json, fill in the app_id and app_secret fields with the correct values.

	In the Facebook developers website, on the Settings page, the website URL needs to read 
	http://ec2-99-79-42-115.ca-central-1.compute.amazonaws.com. Then in the "Advanced" tab, in the 
	"Client OAuth Settings" section, add http://ec2-99-79-42-115.ca-central-1.compute.amazonaws.com and 
	http://99.79.42.115 to the "Valid OAuth redirect URIs" field. Then save these changes.

### The catalog app should now be available at 

      http://99.79.42.115  
      http://ec2-99-79-42-115.ca-central-1.compute.amazonaws.com
