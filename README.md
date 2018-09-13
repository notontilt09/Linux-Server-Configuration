# Linux Server Configuration Guide

## Create Amazon Lightsail Instance
	1.  Use lightsail.aws.amazon.com to configure new Ubuntu server instance

##Login to the instance via SSH

	1. Download Private Key from the Account Page

	2. Copy private key file to `/.ssh/` directory of local machine, and rename `rsakey.pem`

	3. Change permisions on private key so they are not accessible to others by typing `chmod 600 ~/.ssh/rsakey.pem`

	4. Log into Lightsail instance by typing `ssh -i ~/.ssh/rsakey.pem ubuntu@18.206.229.83`


