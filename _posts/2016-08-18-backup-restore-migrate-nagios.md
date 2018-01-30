---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: Migrating Nagios XI to a New Server on Centos6.6
category: ['Centos', 'Nagiosxi', 'Monitoring']
tags: ['centos', 'rhel', 'nagiosxi', 'nagios', 'opensource', 'monitoring']
---

Migrating an old Nagios backup to a new nagiosxi server. Migration is simple in Nagios XI, you a backup and restore it.
Once that is done we need to update/add the clients with IP of the new server, so that both Nagios get alerts. After a while once we are comfortable with the new server we can decommission the old one. 

For example we will use two servers.

* old_nagios (nagiosximon)
* new_nagios (nagioserver)

### Backup from Old Server.

Lets first take a backup from the `old_nagios` server.

	[root@nagiosximon ~]# /usr/local/nagiosxi/scripts/backup_xi.sh 
	Running configuration check...
	Stopping nagios: done.
	Starting nagios: done.
	Backing up Core Config Manager (NagiosQL)...
	tar: Removing leading `/' from member names
	tar: Removing leading `/' from member names
	Backing up Nagios Core...
	tar: Removing leading `/' from member names
	tar: /usr/local/nagios/var/rw/nagios.qh: socket ignored
	tar: /usr/local/nagios/var/ndo.sock: socket ignored
	Backing up Nagios XI...
	tar: Removing leading `/' from member names
	Backing up MRTG...
	tar: Removing leading `/' from member names
	Backing up NRDP...
	tar: Removing leading `/' from member names
	Backing up Nagvis...
	tar: Removing leading `/' from member names
	Backing up MySQL databases...
	Backing up logrotate config files...
	Backing up Apache config files...
	Compressing backup...
	 
	===============
	BACKUP COMPLETE
	===============
	Backup stored in /store/backups/nagiosxi/1471501361.tar.gz
	[root@nagiosximon ~]# ls


### Restoring backup to the `new_nagios` server. 

IMPORTANT : But now most of the server will not be able to send notifications to the new server, so you will recieve notifications, better idea is to disable notifications till we are done with all the configuration.

	[root@nagiosserver ahmed]# /usr/local/nagiosxi/scripts/restore_xi.sh /home/ahmed/Desktop/1471501361.tar.gz 
	TS=1471501690
	Extracting backup to /store/backups/nagiosxi/1471501690-restore...
	In /store/backups/nagiosxi/1471501690-restore/1471501361...
	Backup files look okay.  Preparing to restore...
	Shutting down services...
	Stopping nagios: done.
	Stopping ndo2db: done.
	NPCD Stopped.
	Restoring directories to /...
	Restoring Nagios Core...
	Restoring Nagios XI...
	Restoring NagiosQL...
	Restoring NagiosQL backups...
	Restoring NRDP backups...
	Restoring MRTG...
	Restoring Nagvis backups...
	Restoring MySQL databases...
	Restoring Nagios XI MySQL database...
	Restarting database servers...
	Stopping mysqld:                                           [  OK  ]
	Starting mysqld:                                           [  OK  ]
	Restoring logrotate config files...
	Restoring Apache config files...
	Stopping httpd:                                            [  OK  ]
	Starting httpd:                                            [  OK  ]
	NPCD started.
	Starting ndo2db: done.
	Starting nagios: done.
	 
	===============
	RESTORE COMPLETE
	===============

### Linux Client Configuration [NRPE].

As the service is running on the server, and the inbound requests are handled by `xinetd`, so we need to add the new server information in the configuration. 

* Open file `/etc/xinetd.d/nrpe`
* Added a new entry `only_from	+= 172.3.2.0/24` after the `only_from` line. We have given a mask address, but this can be an IP as well. (This should be the IP Address or IP Range for `new_nagios` server)
* Save, Close and Restart `xinetd`

Here is how the configuration file looks like.

	root@nagiosxi-test-linux:~# cat /etc/xinetd.d/nrpe 
	# default: on
	# description: NRPE (Nagios Remote Plugin Executor)
	service nrpe
	{
	       	flags           = REUSE
		socket_type     = stream    
		port		= 5666    
	       	wait            = no
		user            = nagios
		group		= nagios
	       	server          = /usr/local/nagios/bin/nrpe
		server_args     = -c /usr/local/nagios/etc/nrpe.cfg --inetd
	       	log_on_failure  += USERID
		disable         = no

		# Old server IP
		only_from	= 172.2.2.123

		# New Server IP
		only_from	+= 172.3.2.123
		
		# Range of IPs using Masking.
		only_from 	+= 172.3.2.0/24		
	}

Restart `xinetd` service.

	root@nagiosxi-test-linux:~# service xinetd restart
	xinetd stop/waiting
	xinetd start/running, process 15883


### Windows Client Configuration [NSClient++].

Here as well we have to update allowed hosts parameter in `nsclient.ini`

* Open `run` -> `services.msc`
* Stop the `NSClient++` service.
* Goto Location `C:\Program Files\NSClient++`
* Now go to `start` -> `notepad` -> open notepad with `Run as Administrator` Option.
* Now your notepad has `admin` permission.
* Open the file `nsclient.ini` which is located in `C:\Program Files\NSClient++`
* Added the `new_nagios` server IP in the `allowed hosts` list, its a `comma separated values` list.

Here is how **part** of the configuration looks like. 

	;Undocumented section
	[/settings/default]

	;Undocumented key
	password = nagios123

	;Undocumented key
	allowed hosts = 127.0.0.1, 172.2.2.123, 172.3.2.123

* Finally save, close and Start `NSClient++` service.

We are done. Now you will start seeing updated service information on the `new_nagios` server.

