# Linux Server Configurattion Project

#### By: Sarah Thomens


## IP Address and SSH Port

* Public IP Address: 3.90.7.32
* SSH Port:


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
2. Update all currently installed packages
	* `sudo apt-get update`
	* `sudo apt-get upgrade`

## Third-Party Resources Used to Help

* Udacity has a great course on learning how to make a linux server from scratch. To access, click [here](https://classroom.udacity.com/courses/ud299)
* This project uses [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/instances) to host our server.
