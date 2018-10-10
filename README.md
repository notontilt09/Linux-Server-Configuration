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

	return [output]```

8.  Reload the browser navigating to the Lightstail public IP to see 'Hello World!' displayed.



