---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: RPM Command Cheat Sheet
category: ['Centos', 'Rhel', 'Rpms']
tags: ['centos', 'rhel', 'rpms']
---

RPM (Redhat Package Manager) is the most popular package utility and is used mostly on RHEL, Centos and Fedora. 
RPM helps user/admins to build, install, query, verify, update, and remove/erase individual software packages.

More information can found [rpm.org](http://www.rpm.org/max-rpm/rpm.8.html) and [techmint.com](http://www.tecmint.com/20-practical-examples-of-rpm-commands-in-linux/)

### Install Package

~~~ ruby
rpm -ivh [__rpmfile__]
rpm -ivh vim-common-7.4.629-5.el6.i686.rpm
rpm -ivh --test vim-common-7.4.629-5.el6.i686.rpm
~~~

### Upgrade Package

	rpm -Uvh [__rpmfile__]	
	rpm -Uvh vim-common-7.4.629-5.el6.i686.rpm
	rpm -Uvh --test vim-common-7.4.629-5.el6.i686.rpm

### Erase/Uninstall Package

	rpm -ev [__package_name__]	
	rpm -ev vim-common

### Erase/Uninstall an installed package without checking for dependencies	

	rpm -ev --nodeps [__package_name__]	
	rpm -ev --nodeps vim-common

### List all installed packages

	rpm -qa		
	rpm -qa | grep vim 

### Information along with package version and short description
    
    rpm -qi [__package_name__]		
    rpm -qi vim-common

### Find out what package a file belongs to i.e. find what package owns the file

    rpm -qf [__/path/to/file__]		rpm -qf /etc/passwd
    rpm -qf /bin/bash

### Display list of configuration file(s) for a package   
 
    rpm -qc [__pacakge_name__]
	rpm -qc httpd

### Display list of configuration files for a command
    
    rpm -qcf [__/path/to/file__]		
    rpm -qcf /usr/X11R6/bin/xeyes
 
### Display list of all recently installed RPMs
 
    rpm -qa --last		
    rpm -qa --last
    rpm -qa --last | less


### Find out what dependencies a rpm file has

    rpm -qpR [__rpmfile__]
    rpm -qR [__package_name__]
    
    rpm -qpR vim-common-7.4.629-5.el6.i686.rpm
    rpm -qR bash
    


