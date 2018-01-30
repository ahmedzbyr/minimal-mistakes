---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: Update hosts file in Windows 8
category: ['Windows', 'Hostfile']
tags: ['windows']
---

Host file contains IP followed by the FQDN which can be used to reach that IP address. Host file takes precedence over your DNS servers. In Microsoft operating systems, the HOSTS file is located in the following location: `C:\Windows\System32\Drivers\etc`

You will see a file called `host` below are the contents.


	# Copyright (c) 1993-2009 Microsoft Corp.
	#
	# This is a sample HOSTS file used by Microsoft TCP/IP for Windows.
	#
	# This file contains the mappings of IP addresses to host names. Each
	# entry should be kept on an individual line. The IP address should
	# be placed in the first column followed by the corresponding host name.
	# The IP address and the host name should be separated by at least one
	# space.
	#
	# Additionally, comments (such as these) may be inserted on individual
	# lines or following the machine name denoted by a '#' symbol.
	#
	# For example:
	#
	#      102.54.94.97     rhino.acme.com          # source server
	#       38.25.63.10     x.acme.com              # x client host
	
	# localhost name resolution is handled within DNS itself.
	#	127.0.0.1       localhost
	#	::1             localhost
	
	172.2.2.24  chefmgrserver.ahmed.com

**NOTE this file cannot be updated without an admin permission.**

Here is how you can update this file.

* First go to `start` -> `notepad` -> open notepad with `Run as Administrator` Option.
* Now your notepad has `admin` permission.
* Open the `host` file in the notepad and update the contents as you like.
* Update should be `<IP Address>	<fqdn.ahmed.com>`.
* Save and close.
* Now if you are running a `webserver` or some `web application` on the `IP` address mentioned above, then you can go to the browser and hit `http://<fqdn>` you will reach the `IP` you had mentioned in the `host` file.

### On Linux this is way too simple.

Host file is present in `/etc/hosts`, update using `sudo` privileges and you are done. 
