---
title: sudo Permission On Server - effective uid is not 0, is sudo installed setuid root?
category: ['Linux']
tags: ['linux', 'setuid', 'root', 'permissions']
---

Got into this issue on the server, when trying to install `mysql-server`. 

	[zahmed@ahmed-server ~]$ sudo yum install mysql-server
	sudo: effective uid is not 0, is sudo installed setuid root?
	[zahmed@ahmed-server ~]$ sudo su
	sudo: effective uid is not 0, is sudo installed setuid root?

Checking the permissions from the `root` login, looks messed up. 

	[root@ahmed-server home]# ls -l /usr/bin/sudo
	---x--x--x. 1 root root 123832 Nov 22  2013 /usr/bin/sudo

Setting uid to `sudo` and proper permissions.

	[root@ahmed-server home]# chmod 4755 /usr/bin/sudo

Lets check the permissions now and take it for a spin. 	

	[root@ahmed-server home]# ls -l /usr/bin/sudo
	-rwsr-xr-x. 1 root root 123832 Nov 22  2013 /usr/bin/sudo
	[root@ahmed-server home]# su zahmed
	[zahmed@ahmed-server home]$ sudo su
	[root@ahmed-server home]#

We are all set. Below are few bit setting options.

	# To add the setuid bit
	chmod 4755 /usr/bin/filename
	
	# To set the setgid bit
	chmod 2755 /usr/bin/filename
	
	# To set both the setuid bit and the setgid bit
	chmod 6755 /usr/bin/filename 