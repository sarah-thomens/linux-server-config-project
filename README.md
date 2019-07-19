# Linux Server Configuration Project

#### By: Sarah Thomens


## IP Address and SSH Port

* Public IP Address: 3.90.7.32
* SSH Port: 2200


## URL to Hosted Web Application

* URL:


## Software Installed

* Software GOES HERE


## Configuration Steps

The following steps will take you through how to create your own Amazon Lightsail hosted Ubuntu Linux server using the steps I took to create mine.

1. First, create a new Ubuntu Linux Server Instance
	* [Login or create an account](https://lightsail.aws.amazon.com/) if you don't have one
	* Create an instance
	* Make sure you select <strong>OS only</strong>
	* Choose <strong>Ubuntu</strong> as the instance image - I chose <strong>Ubuntu 16.04 LTS</strong>
	* Choose an instance plan (I chose the cheapest one for this project)
	* Give the instance a host name - I called mine <strong>linux-server-config-project</strong>
	* Wait for it to start and then use it!
2. Follow instructions on page to SSH into the instance
	* To SSH from your own terminal:
		1. Open Account Menu on Amazon Lightsail
		2. Open SSH Keys tab and download the given Default Key. This is your private key: <strong>'LightsailDefaultKey-\*.pem'</strong>
		3. Move the default key file to your computer's local ~/.ssh
			* `mv LightsailDefaultKey-*.pem ~/.ssh`
		4. Move to the ~/.ssh directory
			* `cd ~/.ssh`
		4. Rename the default file
			* `mv LightsailDefaultKey-*.pem udacity_key.rsa`
		5. Change the file permissions for the private key to read/write
			* `chmod 600 ~/.ssh/udacity_key.rsa`
		6. Finally, connect to the Lightsail server
			* `ssh -i ~/.ssh/udacity_key.rsa ubuntu@3.90.7.32`
			* 3.90.7.32 is the public IP address for the server
			* All following commands should be run from the server terminal unless otherwise specified.
3. Update all currently installed packages
	* `sudo apt-get update`
	* `sudo apt-get upgrade`
4. Configure the Lightsail Firewall
	* Change the SSH port from 22 to 2200 and make sure to allow it in the firewall.
		1. Edit <strong>sshd_config</strong> file
			* `sudo nano /etc/ssh/sshd_config`
		2. On Line 5, where it says Port 22, change to Port 2200
			* This will help add an additional layer of security by changing the port from the default.
		3. Find the line PermitRootLogin and edit from saying <strong>prohibit-password</strong> to <strong>no</strong>
			* This prevents users from logging in as the root and gaining access to the entire server.
		4. Save the file with <strong>CTRL + O</strong>
		5. Exit the file with <strong>CTRL + X</strong>
		6. Force the server to restart the SSH connection and use these new settings:
			* `sudo service ssh restart`
	* Configure the Firewall to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
		1. To check that the status of the firewall is <strong>inactive</strong>
			* `sudo ufw status`
		2. To make the configuration easier, begin by denying all incoming messages and allowing all outgoing messages by default
			* `sudo ufw default deny incoming`
			* `sudo ufw default allow outgoing`
		3. Allow the individual ports for SSH, HTTP, and NTP
			* `sudo ufw allow 2200/tcp`
			* `sudo ufw allow www`
			* `sudo ufw allow 123/udp`
		4. Deny the default port 22
			* `sudo ufw deny 22`
		5. Enable the Firewall
			* `sudo ufw enable`
			* It will prompt you saying it could interrupt ssh connections. Type <strong>y</strong>
			* Exit the server: `exit`
		6. Configure the Firewall Settings on Amazon Lightsail
			* Open the instance of your server
			* Click the Networking tab within the instance
			* Change the Firewall rules to follow what was done in the terminal
			* HTTP Port 80 stays the same
			* Deny Port 22 labelled as SSH
			* Create a custom rule with protocol TCP and port 2200 for the new SSH port
			* Create a custom rule with protocol UDP and port 123 for the NTP port
		7. Restart the server from your terminal to make sure the firewall still allows SSH incoming
			* `ssh -i ~/.ssh/udacity_key.rsa -p 2200 ubuntu@3.90.7.32`
	* Give Grader Access to Server
		1. Running the server from your terminal, create a new user named grader
			* `sudo adduser grader`
			* Type an initial password (this will not be used later, so make it something you can remember initially)
			* Follow the prompts and give the user a name and other information
		2. Give the new user sudo capabilities
			* `sudo visudo` - this opens the sudoers file for you to edit
			* Find the line that says <strong>root ALL=(ALL:ALL) ALL</strong> and add below it the line <strong>grader ALL=(ALL:ALL) ALL</strong>
				* This adds the new user to the list of sudoers
			* Save the changes: `CTRL + O`
			* Exit the file: `CTRL + X`
		3. Create an SSH key pair for the new grader user using the ssh-keygen tool
			* Open a new terminal running from your local machine
			* Change to the /.ssh directory: `cd ~/.ssh`
			* Create a new keygen file: `ssh-keygen -f ~/.ssh/grader_key.rsa`
			* If you want to create a passphrase for the new user, do this here, but it is not necessary to do so. Just press enter twice without typing anything to skip the passphrase.
			* Now we need to copy the key from the new file
				* Display the file contents: `cat ~/.ssh/grader_key.rsa.pub`
				* Copy what is displayed
			* Go back to the original ubuntu terminal create an authorized_keys file
				* `cd /home/grader`
				* If there is no .ssh directory, create one: `sudo mkdir .ssh`
				* `cd .ssh`
				* Create the authorized_keys file: `sudo touch authorized_keys`
				* Open the file to edit: `sudo nano authorized_keys`
				* Paste what was in the grader_key.rsa.pub file into the authorized_keys file
				* Save the file: `CTRL + O`
				* Exit the file: `CTRL + X`
			* Change the permissions of the files to grader
				* `sudo chmod 700 /home/grader/.ssh` - set the .ssh directory permissions so only grader can access
				* `sudo chmod 644 /home/grader/.ssh/authorized_keys` - allowed the grader to read and write authorized_keys and everyone else to only read
				* `sudo chown -R grader:grader /home/grader/.ssh` - changes the owner of the .ssh directory to grader
				* `sudo service ssh restart` - restart the server





## Third-Party Resources Used to Help

* Udacity has a great course on learning how to make a linux server from scratch. To access, click [here](https://classroom.udacity.com/courses/ud299)
* This project uses [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/instances) to host our server.
