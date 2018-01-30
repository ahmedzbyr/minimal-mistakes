---
title: Creating a passwordless entry on Centos 6.5.
category: ['Linux']
tags: ['linux', 'centos', 'rhel']
---

You can enter a linux system without entring a password using below steps.
We will be creating a ssh jey which will be share between the servers, which will be used to authenticate. This is also a secure way of connecting to server, as the private key is inside the user home directory and can only be accessed by the user.

In the exmaple below we are make connecting to `SERVER02` and `SERVER03` passwordless from `SERVER01`.

    SERVER01 ----(no password) ----- > SERVER02
                       |
                        ------------ > SERVER03

###  Creating passwordless entry from (`SERVER01`) to other servers.

Create a `rsa` key on `SERVER01`

	ssh-keygen -t rsa
	
Create `.ssh` directory on other 2 servers.
	
	ssh ahmed@SERVER02 mkdir -p .ssh
	ssh ahmed@SERVER03 mkdir -p .ssh
	
Add the `id_rsa.pub` to `authorized_keys`
	
	cat ~/.ssh/id_rsa.pub | ssh ahmed@SERVER02 'cat >> .ssh/authorized_keys'
	cat ~/.ssh/id_rsa.pub | ssh ahmed@SERVER03 'cat >> .ssh/authorized_keys'

Make sure we have the right permissions.
	
	ssh ahmed@SERVER02 chmod 744 -R .ssh 
	ssh ahmed@SERVER03 chmod 744 -R .ssh 

Testing.	

	ssh ahmed@SERVER02
	ssh ahmed@SERVER03
	
