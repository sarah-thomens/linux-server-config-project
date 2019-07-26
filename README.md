# Linux Server Configuration Project

#### By: Sarah Thomens


## IP Address and SSH Port

* Public IP Address: 3.90.7.32
* SSH Port: 2200


## URL to Hosted Web Application

* URL:


## Software Installed

* Apache
* Mod-WSGI
* Git
* python-setuptools
* virtualenv
* flask
* httplib2
* requests
* oauth2client
* SQLAlchemy
* Python PostgreSQL adaptor psycopg



## Configuration Steps

The following steps will take you through how to create your own Amazon Lightsail hosted Ubuntu Linux server using the steps I took to create mine.


#### Create a New Ubuntu Linux Server Instance
1. [Login or create an account](https://lightsail.aws.amazon.com/), if you don't have one
2. Create an instance
3. Make sure you select <strong>OS only</strong>
4. Choose <strong>Ubuntu</strong> as the instance image
	* I chose <strong>Ubuntu 16.04 LTS</strong>
5. Choose an instance plan
	* I chose the cheapest option for this project, free for the first month
6. Give the instance a host name
	* I called mine <strong>linux-server-config-project</strong>
7. Wait for it to start and then use it!


#### Adapt Server to be able to SSH from own Terminal
1. Open the Account Menu on Amazon Lightsail
2. Open SSH Keys tab and download the given Default Key. This is your private key.
3. Move the private key file to your computer's local ~/.ssh
	* `mv <PRIVATE-KEY-FILE-NAME-HERE> ~/.ssh`
4. Move to the ~/.ssh directory
	* `cd ~/.ssh`
5. Rename the default file
	* `mv <PRIVATE-KEY-FILE-NAME-HERE> udacity_key.rsa`
6. Change the file permissions for the private key to read/write
	* `chmod 600 ~/.ssh/udacity_key.rsa`
7. Finally, connect to the Lightsail server
	* `ssh -i ~/.ssh/udacity_key.rsa ubuntu@<YOUR-PUBLIC-IP-ADDRESS>`
	* 3.90.7.32 is the public IP address for my server, use your own here
	* All following commands should be run from the server terminal unless otherwise specified.


#### Update All Currently Installed Packages
* `sudo apt-get update`
* `sudo apt-get upgrade`


#### Configure the Lightsail Firewall
1. Change the SSH port from 22 to 2200 and make sure to allow it in the firewall.
	* Edit <strong>sshd_config</strong> file: `sudo nano /etc/ssh/sshd_config`
	* Where it says Port 22, change to Port 2200
		* This will help add an additional layer of security by changing the port from the default.
	* Find the line PermitRootLogin and edit the line <strong>prohibit-password</strong> to <strong>no</strong>
		* This prevents users from logging in as the root and gaining access to the entire server.
	* Save the file with <strong>CTRL + O</strong>
	* Exit the file with <strong>CTRL + X</strong>
	* Force the server to restart the SSH connection and use these new settings:
		* `sudo service ssh restart`
2. Configure the Firewall to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
	* To check that the status of the firewall is <strong>inactive</strong>
		* `sudo ufw status`
	* To make the configuration easier, begin by denying all incoming messages and allowing all outgoing messages by default
		* `sudo ufw default deny incoming`
		* `sudo ufw default allow outgoing`
	* Allow the individual ports for SSH, HTTP, and NTP
		* `sudo ufw allow 2200/tcp`
		* `sudo ufw allow www`
		* `sudo ufw allow 123/udp`
	* Deny the default port 22
		* `sudo ufw deny 22`
	* Enable the Firewall
		* `sudo ufw enable`
		* It will prompt you saying it could interrupt ssh connections. Type <strong>y</strong>
		* Exit the server: `exit`
3. Configure the Firewall Settings on Amazon Lightsail
	* Open the instance of your server
	* Click the Networking tab within the instance
	* Change the Firewall rules to follow what was done in the terminal:
		* HTTP Port 80 stays the same
		* Deny Port 22 labelled as SSH
		* Create a custom rule with protocol TCP and port 2200 for the new SSH port
		* Create a custom rule with protocol UDP and port 123 for the NTP port
4. Restart the server from your terminal to make sure the firewall still allows SSH incoming
	* `ssh -i ~/.ssh/udacity_key.rsa -p 2200 ubuntu@3.90.7.32`


#### Give Grader Access to Server
1. Create a new user named grader
	* `sudo adduser grader`
	* Type an initial password - this will not be used later, so make it something you can remember initially
	* Follow the prompts and give the user a name and other information
2. Give the new user sudo capabilities
	*  Open the sudoers file for you to edit
		* `sudo visudo`
	* Find the line: <strong>root ALL=(ALL:ALL) ALL</strong>
	* Add below it: <strong>grader ALL=(ALL:ALL) ALL</strong>
	* This adds the new user to the list of sudoers
	* Save the changes: `CTRL + O`
	* Exit the file: `CTRL + X`
	* List all users and make sure your user was added
		* `cut -d: -f1 /etc/passwd`
3. Create an SSH key pair for the new grader user using the ssh-keygen tool
	* Open a new terminal running from your local machine
	* Change to the /.ssh directory
		* `cd ~/.ssh`
	* Create a new keygen file
		* `ssh-keygen -f ~/.ssh/grader_key.rsa`
	* If you want to create a passphrase for the new user, do this here, but it is not necessary to do so. Just press enter twice without typing anything to skip the passphrase.
4. Copy the key from the new file to the authorized_keys file
	* Display the file contents
		* `cat ~/.ssh/grader_key.rsa.pub`
	* Copy what is displayed
	* Go back to the original ubuntu terminal, create an authorized_keys file
		* `cd /home/grader`
		* If there is no .ssh directory, create one: `sudo mkdir .ssh`
		* `cd .ssh`
		* Open the file to edit: `sudo nano authorized_keys`
		* Paste what was in the grader_key.rsa.pub file into the authorized_keys file
		* Save the file: `CTRL + O`
		* Exit the file: `CTRL + X`
5. Change the permissions of the files to grader
	* Set the .ssh directory permissions so only grader can access
		* `sudo chmod 700 /home/grader/.ssh`
	* Allow the grader to read and write authorized_keys and everyone else to only read
		* `sudo chmod 644 /home/grader/.ssh/authorized_keys`
	* Change the owner of the .ssh directory to grader
		* `sudo chown -R grader:grader /home/grader/.ssh`
6. Restart the Server
	* `sudo service ssh restart`


#### Configure the local timezone to be UTC
1. `sudo dpkg-reconfigure tzdata`
2. Choose the option None of the above
3. Choose the option UTC


#### Install and configure Apache to serve a Python mod_wsgi application.
1. Install Apache
	* `sudo apt-get install apache2`
2. To make sure Apache installed properly, type your public IP address(3.90.7.32) into your address bar and the Apache2 Ubuntu Default Page should appear
3. Install the application handler mod_wsgi and python helper package python-setuptools
	* <strong>(WARNING - DO NOT DO BOTH STEPS - having two mod_wsgi packages installed will cause errors, choose one for a Python 2.7 OR Python 3 project)</strong>
		* For projects using Python 2.7
			* Download and install python 2.7
				* `sudo apt-get install python2.7 python-pip`
				* Do not uninstall Python 3, this will cause server issues
			* Download and install the application handler mod_wsgi and python-setuptools
				* `sudo apt-get install python-setuptools libapache2-mod-wsgi`
		* For projects using Python 3
			* Download and install the application handler mod_wsgi and python-setuptools
				* `sudo apt-get install python-setuptools libapache2-mod-wsgi-py3`
4. To make sure the application handler is enabled
	* `sudo a2enmod wsgi`
5. Restart the server with the new module
	* `sudo service apache2 start`


#### Install Git on the server
* `sudo apt-get install git`


#### Setup the Server to Deploy a Flask App
1. Change your directory to where the project will be placed
	* `cd /var/www`
2. Make a new directory for your project
	* `sudo mkdir catalog`
3. Change the owner of the directory to the grader
	* `sudo chown -R grader:grader catalog`
4. Move inside your new directory
	* `cd catalog`
5. Make another directory for your project
	* `sudo mkdir catalog`
6. Move inside this directory
	* `cd catalog`
7. Make directories for the Flask project
	* `sudo mkdir static templates`
8. Create a Test App to make sure that your server is running properly
	* `sudo nano __init__.py`
	* Add the following code:
	```
	from flask import Flask

	app = Flask(__name__)

	@app.route("/")
	def hello():
		return "Hello World!"

	if __name__ == "__main__":
		app.run()
	```
9. Set up a virtual environment to keep the application and its dependencies isolated from the main system.
	* Install virtualenv
		* `sudo pip install virtualenv`
	* Create your new virtual environment
		* `sudo virtualenv venv`
	* Change permissions to the virtual environment folder so all parties have read, write, and execute privileges
		* `sudo chmod -R 777 venv`
	* Activate the new environment
		* `source venv/bin/activate`
10. Install Flask so your Flask project will run properly
	* `pip install Flask`
11. Test Run the App to see if there are any errors
	* `python __init__.py`
	* You should see something like <strong>Running on http://127.0.0.1:5000/</strong>
	* Close the application
		* `CTRL + C`
	* Deactivate the Virtual Environment
		* `deactivate`
12. Configure and Enable a new Virtual Host for your Project
	* First you need to find your host name
		* Go [here](https://www.hcidata.info/host2ip.cgi) to find your host name
		* Type your IP Address in the box provided and click enter
		* Your host name should appear with some other information
		* You will use this host name in the following file under ServerAlias
	* Create a virtual host configuration file
		* `sudo nano /etc/apache2/sites-available/catalog.conf`
	* The file should look like this:
		```
		<VirtualHost *:80>
			ServerName 3.90.7.32
			ServerAlias ec2-3-90-7-32.compute-1.amazonaws.com
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
	* Enable the newly created virtual host: `sudo a2ensite catalog`
13. Make a catalog.wsgi file to allow the application to be served over the mod_wsgi
	* Change to the proper directory
		* `cd /var/www/catalog`
	* Create the .wsgi file
		* `sudo nano catalog.wsg`
	* The file should look like this:
		```
		import sys
		import logging
		logging.basicConfig(stream=sys.stderr)
		sys.path.insert(0, "/var/www/catalog/")

		from catalog import app as application
		```
14. Restart Apache to make sure all changes are being used
	* `sudo service apache2 restart`
15. Check to see if your simple project is running by typing your public IP address(3.90.7.32) into your browser


#### Cloning GitHub Project to the Server
1. Clone your project repository from your Github account
	* Change directory into project directory
		* `cd /var/www/catalog/catalog`
	* Clone your project from GitHub
		* `git clone https://github.com/sarah-thomens/item-catalog-project.git`
	* Organize so that all files and directories from git project are directly in the second tier catalog directory
2. Make sure your application.py ends with app.run()
	* Change directories to the proper catalog tier
		* `cd /var/www/catalog/catalog`
	* Open the application.py file
		* `sudo nano application.py`
	* Go to the end of the file and find:
	```
	if __name__ == "__main__":
		app.run(host='0.0.0.0', port=8000)
	```
	* Delete what is within the parentheses after app.run to look like:
	```
	if __name__ == "__main__":
		app.run()
	```


#### Installing the Project's Dependencies
1. Activate the Virtual Environment
	* `source venv/bin/activate`
2. Install the httplib2 module
	* `pip install httplib2`
3. Install the requests module
	* `pip install requests`
4. Install the oauth2client
	* `sudo pip install --upgrade oauth2client`
5. Install SQLAlchemy
	* `sudo pip install sqlalchemy`
6. Install the Python PostgreSQL adaptor psycopg
	* `sudo apt-get install python-psycopg2`


#### Install and Configure PostgreSQL
1. Install PostgreSQL
	* `sudo apt-get install postgresql postgresql-contrib`
2. Check to make sure no remote connections are allowed, this is the default
	* `sudo nano /etc/postgresql/9.5/main/pg_hba.conf`
	* Edit to look like this:
		```
		local   all             postgres                                peer
		local   all             all                                     peer
		host    all             all             127.0.0.1/32            md5
		host    all             all             ::1/128                 md5
		```
3. Make sure you are in the project directory
	* `cd /var/www/catalog/catalog`
4. Open your project's database setup file
	* `sudo nano database_setup.py`
5. Find the line beginning with <strong>engine</strong> and change to:
	* <strong>engine = create_engine('postgresql://catalog:password@localhost/catalog')</strong>
	* Save the file: `CTRL + O`
	* Close the file: `CTRL + X`
6. Open your project's application file
	* `sudo nano application.py`
7. Once again, find the line beginning with <strong>engine</strong> and change it to the same line as in the database_setup file:
	* <strong>engine = create_engine('postgresql://catalog:password@localhost/catalog')</strong>
	* Save the file: `CTRL + O`
	* Close the file: `CTRL + X`
8. Remove the test file you created to test the server
	* `sudo rm __init__.py`
9. Change the name of the application.py file to __init__.py
	* `sudo mv application.py __init__.py`
10. Create needed linux user for psql:
	* `sudo adduser catalog`
	* Choose a password
11. When installing, Postgre creates a user that can access the database system. Switch to this user
	* `sudo su - postgres`
12. Connect to the database system
	* `psql`
13. Create a new user for your project with a password
	* `CREATE USER catalog WITH PASSWORD 'password';`
	* The password should be the same as the engine lines we changed in application.py and database_setup.py
14. Give the catalog user the ability to create databases
	* `ALTER USER catalog CREATEDB;`
15. Check current roles and their attributes
	* `\du`
16. Create a database owned by the catalog user
	* `CREATE DATABASE catalog WITH OWNER catalog;`
17. Connect to the newly created database
	* `\c catalog`
18. Revoke all of the rights from general users
	* `REVOKE ALL ON SCHEMA public FROM public;`
19. Make permissions so only catalog user can create tables
	* `GRANT ALL ON SCHEMA public TO catalog;`
20. Logout of PostgreSQL: `\q`
21. Return to your ubuntu command line: `exit`
22. Set up the database by running your database_setup.py file:
	* `python /var/www/catalog/catalog/database_setup.py`


## Third-Party Resources Used to Help

* Udacity has a great course on learning how to make a linux server from scratch. To access, click [here](https://classroom.udacity.com/courses/ud299)
* This project uses [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/instances) to host our server.
* This website helped greatly when trying to deploy the Flask Application in the Ubuntu server.
	* [DigitalOcean: How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* I used the help of several other Udacity students' projects to piece together my own.
	* [stueken](https://github.com/stueken/FSND-P5_Linux-Server-Configuration)
	* [kcalata](https://github.com/kcalata/Linux-Server-Configuration/blob/master/README.md)
	* [iliketomatoes](https://github.com/iliketomatoes/linux_server_configuration)
