---
title: Installing `python 2.7.x` on Centos 6.5/6.6
category: ['Linux', 'Python', 'Centos', 'Install']
tags: ['python', 'centos', 'linux', 'install']
---

By default centos comes with python 2.6. In most of the cases we might need python 2.7 or later to be installed. Below are few ways to install python 2.7 on centos 6.x. 

##  Method 1 - Using SCL (centos 6.5 and earlier)

First off, update the system to the latest state. Just in case there are any pending fixes.

	yum -y update

####  Next install the `SCL` [Softwate Collection repository](https://wiki.centos.org/AdditionalResources/Repositories/SCL) for centos. 

	yum install centos-release-SCL 
	yum install python27

####  Setting the default for the shell.

	scl enable python27 bash

This will set all the path required.

####  Installing `pip`.

	wget https://bootstrap.pypa.io/get-pip.py
	sudo python27 get-pip.py

##  Method 2 - Manual Install (For Centos version earlier than 6.5).

First off, update the system to the latest state. Just in case there are any pending fixes. 

	yum -y update
	
####  Install Dev Tools

	yum groupinstall -y 'development tools'
	yum install -y zlib-devel openssl-devel sqlite-devel bzip2-devel

####  Download Python, Configure, build and Install.

	wget https://www.python.org/ftp/python/2.7.8/Python-2.7.8.tgz  
	tar -xvzf Python-2.7.8.tgz

Configure and build python and Install.

	cd Python-2.7.8
	./configure --prefix=/usr/local
	make

Install `python`.

	make altinstall  

####  Setting path and Installing `pip`

	#  Example: export PATH="[/path/to/installation]:$PATH"
	export PATH="/usr/local/bin:$PATH"

Installing `pip`.

	wget https://bootstrap.pypa.io/get-pip.py
	sudo python27 get-pip.py

More details in the link [here](https://www.digitalocean.com/community/tutorials/how-to-set-up-python-2-7-6-and-3-3-3-on-centos-6-4)


