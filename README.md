# Linux Server Configuration Guide

## Create Amazon Lightsail Instance
1.  Use lightsail.aws.amazon.com to configure new Ubuntu server instance

## Login to the instance via SSH
1. Download Private Key from the Account Page

2. Copy private key file to `/.ssh/` directory of local machine, and rename `rsakey.pem`

3. Change permisions on private key so they are not accessible to others by typing `chmod 600 ~/.ssh/rsakey.pem`

4. Log into Lightsail instance by typing `ssh -i ~/.ssh/rsakey.pem ubuntu@18.206.229.83`

## Update all currently installed packages.

1. Run `sudo apt-get update`

2. Run `sudo apt-get upgrade`

## Change SSH port from 22 to 2200

1. First update the Lightsail firewall to allow connections to port 2200 by adding it to the list on the networking tab of the Lightsail page

2. In terminal, logged into server, run `sudo nano /etc/ssh/sshd_config`

3. Change port from 22 to 2200

4. Under `#Authentication`, change `PermitRootLogin` to `no` to disable the ability to login as the root user.

5. Save and Exit the file then restart sshd with `sudo service sshd restart`

6. Can now login to the server with `ssh -i ~/.ssh/rsakey.pem -p 2200 ubuntu@18.206.229.83`

## Setup the firewall

Configure the firewall to only accept connections for SSH on port 2200, HTTP on port 80, and NTP on port 123

	sudo ufw allow 2200/tcp
	sudo ufw allow www
	sudo ufw allow 123/udp
	sudo ufw enable

## Create user named grader with sudo access

1. run `sudo adduser grader`

2. run `sudo visudo`

3. Edit the sudo file and add `grader ALL=(ALL:ALL) NOPASSWD:ALL` under the line `root ALL=(ALL:ALL)`

4. Grader user can now sudo with no password requirement.

## Create SSH keypair to login as grader

1. From home computer terminal, run `ssh-keygen`

2. Save file to `/Users/Jooooooooce/.ssh/graderkey` and add passphrase 'grader'

3. In server terminal, navigate to `/home/grader`

4. Create .ssh directory and key file with `sudo mkdir .ssh` and `sudo touch .ssh/authorized_keys`

5. Copy contents of graderker.pub from local computer to the authorized_key file on the server

6. Restart SSH service with `sudo service ssh restart`

7. Can now log in as grader from terminal with `ssh -i ~/.ssh/graderkey grader@18.206.229.83 -p 2200`

## Configure the local timezone to UTC

1. Run `sudo dpkg-reconfigure tzdata`

2. Select UTC from the list of available time zones.

## Configure Apache to serve a Python mod_wsgi application

1. Install apache `sudo apt-get install apache2`

2. In local browser, navigate to the public IP address provided by the Amazon Lightsail instance.  The 'Apache2 Ubuntu Default Page' should show up now.

3. Install mod_wsgi `sudo apt-get install libapache2-mod-wsgi python-dev`

4. Enable mod_wsgi `sudo a2enmod wsgi`

5. Edit the `/etc/apache2/sites-enabled/000-default.conf` file to tell apache how to respond to requests, and where to find the files for a particular site.

At the end of the `<VirtualHost *:80>` block, add the following line `WSGIScriptAlias / /var/www/html/myapp.wsgi`

6. Restart Apache with `sudo apache2ctl restart`

7. Create the myapp.wsgi file using the command `sudo nano /var/www/html/myapp.wsgi` and add the following code to deploy a simple application displaying 'Hello World' to the user
```def application(environ, start_response):
	status = '200 OK'
	output = 'Hello World!'

	response_headers = [('Content-type', 'text/plain'), ('Content-Length', str(len(output)))]
	start_response(status, response_headers)

	return [output]
```

8.  Reload the browser navigating to the Lightstail public IP to see 'Hello World!' displayed.

## Install Git

1. Run `sudo apt-get install git`

## Deploy Flask application

1. Move to the /var/www directory `cd /var/www`

2. Create the directory structure to be used for the project with the following commands
```
sudo mkdir catalog
cd catalog
sudo mkdir catalog
cd catalog
sudo mkdir static templates
```
The directory structure will look like the following:
```
---catalog
------catalog
---------static
---------templates
```

3. Create the __init__.py file that will contain the flask application logic `sudo nano __init__.py`

	Add the following logic to the file:
	```
	from flask import Flask
	app = Flask(__name__)
	@app.route("/")
	def hello():
		return "Hello, There!"
	if __name__ == "__main__":
		app.run()
		```
Save and close the file.

4.  Install Flask inside a virtual environment to run the flask application:
```
sudo apt-get install python-pip
sudo pip install virtualenv
sudo virtualenv venv
source venv/bin/activate
sudo pip install Flask
```

5.  Check if the installation is successfull and the app in running:
`sudo python __init__.py`
	If successful, it will display "Running on http://127.0.0.1:5000/"

6.  Use CTRL+C to quit the Flask app and then `deactivate` to deactivate the virtual environment.

7. Configure and Enable a New Virtual Host
`sudo nano /etc/apache2/sites-available/catalog.conf`

  Add the following code using the Amazon Lightsail Public IP as the ServerName:

  ```
  <VirtualHost *:80>
		ServerName 54.173.178.203
		ServerAdmin notontilt09@gmail.com
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

8. Create the .wsgi file:
```
cd /var/www/catalog
sudo nano catalog.wsgi
```
  Add the following code to the catalog.wsgi file:
 ```
 #!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'Add your secret key'
```

The directory structure should look like this:
```
---catalog
------catalog
---------static
---------templates
---------venv
---------__init__.py
------catalog.wsgi
```

9. Restart Apache with `sudo service apache2 restart`

10. The browser navigating to the public IP should now display the `__init__.py` file.



## Clone item catalog repository

1. In the `catalog` directory, clone the item catalog repository with `sudo git clone https://github.com/notontilt09/fortnite.git`

## Install necessary Flask and SQL packages in the virtual environment

1. In the `catalog` folder containing the `venv` virtual environment, install the followwing packages that are used in the item catalog python files
```
sudo pip install requests
sudo pip install httplib2
sudo pip install oauth2client
sudo pip install sqlalchemy
sudo pip install Flask-SQLAlchemy
sudo pip install passlib
```
## Change item catalog python files from sqlite to postgresql

1. Three files are using the sqlite database schema to serve the data:  `fortnite_database_setup.py`, `fortnite_catalog.py`, and `add_weapons.py`.  In each of these files, change the line:
`engine = create_engine('sqlite:fortniteweapondatabase.db')` to `engine = create_engine('postgresql://catalog:catalog-pw@localhost/catalog')`

## Install Postgresql and set up database

1. Install psycopg2 `sudo apt-get install python-psycopg2`

2. Install Postgresql `sudo apt-get install postgresql postgresql-contrib`

3. Change to the postgres user `sudo su - postgres`

4. Open postgresql `psql`

5. Create catalog user and give it the correct access
```
CREATE USER catalog WITH PASSWORD 'catalog-pw';
ALTER USER catalog CREATEDB;
CREATE DATABASE catalog WITH OWNER catalog;
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
\q
exit
```
This will create the database with the correct permissions established.  We can now run the item catalog python files to fill this database with data.

## Update the `__init__.py` file to correctly locate the `client_secrets.json` file

1. In `__init__.py` change the line `CLIENT_ID = json.loads( open('client_secrets.json', 'r').read())['web']['client_id']` to `CLIENT_ID = json.loads(open(r'/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']`

## Run the program

1. Run `python fortnite_database_setup.py` to setup the postgreSQLdatabase

2. Run `python add_weapons.py` to add the information to the postgreSQL database

3. Run 'python __init__.py' to run the actual application

4. The application should now run on `http://http://54.173.178.203/`.




## Third Party References

1. https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

2. Udacity Linux Server Configuration course and discussion forums.