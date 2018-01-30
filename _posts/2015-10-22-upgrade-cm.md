---
title: Update Cloudera Manager to specific version [5.4.5]
category: ['Linux', 'Hadoop']
tags: ['linux', 'hadoop', 'upgrade', 'cloudera', 'manager']
---

Upgrading Cloudera Manager 5 to the Latest Cloudera Manager, In most cases it is possible to complete the following upgrade without shutting down most CDH services, although you may need to stop some dependent services. CDH daemons can continue running, unaffected, while Cloudera Manager is upgraded. The upgrade process does not affect your CDH installation. After upgrading Cloudera Manager you may also want to upgrade CDH 5 clusters to CDH 5.4.5 or latest.

###  Take database backup.

If we are running a dedicated database which is recommended in production setup. then we need to take a backup of the DB as a precaution. 

Assuming we are using a dedicated DB.

###  Stop Cloudera Manager Server, Database, and Agent

Shutdown cloudera manager server.

	sudo service cloudera-scm-server stop

If cloudera manager is also running an Agent service. 

	sudo service cloudera-scm-agent stop

NOTE : If we are using a standalone database then we need to stop that as well.
	
	sudo service cloudera-scm-server-db stop


##  Update repository to get the latest rpm.

Create a file `cloudera-manager.repo` with below contents.

	[cloudera-manager]
	#  Packages for Cloudera Manager, Version 5.4.5, on RedHat or CentOS 6 x86_64
	name=Cloudera Manager
	baseurl=http://archive.cloudera.com/cm5/redhat/6/x86_64/cm/5.4.5/
	gpgkey=http://archive.cloudera.com/cm5/redhat/6/x86_64/cm/RPM-GPG-KEY-cloudera 
	gpgcheck=1

copy `cloudera-manager.repo` to `/etc/yum.repos.d/`


	$ sudo yum clean all
	$ sudo yum upgrade cloudera-manager-server cloudera-manager-daemons cloudera-manager-agent

Check if all the installation was good.

	$ rpm -qa 'cloudera-manager-*'
	cloudera-manager-repository-5.0-1.noarch
	cloudera-manager-server-5.4.7-0.cm544.p0.932.el6.x86_64
	cloudera-manager-server-db-2-5.4.7-0.cm544.p0.932.el6.x86_64
	cloudera-manager-agent-5.4.7-0.cm544.p0.932.el6.x86_64
	cloudera-manager-daemons-5.4.7-0.cm544.p0.932.el6.x86_64

###  Start the Cloudera Manager Server (Packages)

	sudo service cloudera-scm-server start
	sudo service cloudera-scm-agent start
	
##  Upgrade CDH version from 5.4.2 to 5.4.5 

###  Manually adding - Parcel to Repository

> * Download and copy `parcel` to `/opt/cloudera/parcel-repo` on cloudera Manager.
	
	wget http://archive.cloudera.com/cdh5/parcels/5.4.5/CDH-5.4.5-1.cdh5.4.5.p0.7-el6.parcel`

> * Download the `sha` file and copy to `/opt/cloudera/parcel-repo` directory.

	wget http://archive.cloudera.com/cdh5/parcels/5.4.5/CDH-5.4.5-1.cdh5.4.5.p0.7-el6.parcel.sha1

> * Change permission to both files above to `cloudera-scm`.
> * Rename file which has the `shasum` from  `CDH-5.4.5-1.cdh5.4.5.p0.7-el6.parcel.sha1` to `CDH-5.4.5-1.cdh5.4.5.p0.7-el6.parcel.sha`.
> * Now check on Cloudera Manager portal  for the new parcel. 
> * Here is the link to the parcel http://cloudera-manager-server:7180/cmf/parcel/status 
> * It will take 5-10min to update the list of parcels. Depending on the refresh frequency.  
> * Once it is does we will see a `Distruibute` button.
> * Click `Distruibute` and then `Active`.
> * Next go to `home` and click on `cluster` and select `upgrade cluster`, follow the instructions.
> * Restart the cluster. Do a `Rolling restart`.
> * We are done.
