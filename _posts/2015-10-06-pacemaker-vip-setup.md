---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: Creating a two-node RHEL cluster with Virtual IP using CMAN and Pacemaker.
category: ['Linux', 'Ha']
tags: ['linux', 'ha', 'high-availability', 'cman', 'pacemaker', 'heartbeat', 'haproxy']
---

CMAN v3 is a Corsync plugin that monitors the names and number of active cluster nodes in order to deliver membership and quorum information to clients (such as the Pacemaker daemons). We are using this to have a VIP (Virtual IP) being made high available.

Links : 
	
[Blog Matt](http://blog.mattbrock.co.uk/creating-a-two-node-centos-6-cluster-with-floating-ip-using-cman-and-pacemaker/)

[Cluster Labs](http://clusterlabs.org/quickstart-redhat-6.html)

	
###  Configuring Repo on RHEL 6.6
	
	[root@waepprrkhe001 ~]# cat /etc/yum.repos.d/centos.repo
	[centos-6-base]
	name=CentOS-$releasever - Base
	mirrorlist=http://mirrorlist.centos.org/?release=6&arch=x86_64&repo=os
	baseurl=http://mirror.centos.org/centos/6/os/x86_64/
	enabled=1
	gpgkey=http://mirror.centos.org/centos/6/os/x86_64/RPM-GPG-KEY-CentOS-6
	[root@waepprrkhe001 ~]#

	
###  Installation and initial configuration

Install the required packages on both machines:

	yum install pacemaker cman pcs ccs resource-agents

Set up and configure the cluster on the primary machine, changing newcluster , primary.server.com and secondary.server.com as needed:

	ccs -f /etc/cluster/cluster.conf --createcluster newcluster
	ccs -f /etc/cluster/cluster.conf --addnode primary.server.com
	ccs -f /etc/cluster/cluster.conf --addnode secondary.server.com
	ccs -f /etc/cluster/cluster.conf --addfencedev pcmk agent=fence_pcmk
	ccs -f /etc/cluster/cluster.conf --addmethod pcmk-redirect primary.server.com
	ccs -f /etc/cluster/cluster.conf --addmethod pcmk-redirect secondary.server.com
	ccs -f /etc/cluster/cluster.conf --addfenceinst pcmk primary.server.com pcmk-redirect \
                                                                        port=primary.server.com
	ccs -f /etc/cluster/cluster.conf --addfenceinst pcmk secondary.server.com pcmk-redirect \
                                                                        port=secondary.server.com

Copy `/etc/cluster/cluster.conf` from the primary server to secondary server in cluster.

It's necessary to turn off quorum checking, so do this on both machines:

	echo "CMAN_QUORUM_TIMEOUT=0" >> /etc/sysconfig/cman

###  Start the services

Start up the services on both servers. 

	service cman start
	service pacemaker start
	
Make sure both services can be reboot:

	chkconfig cman on
	chkconfig pacemaker on
	
###  Configure and create floating IP

Configure the cluster on the primary server.

	pcs property set stonith-enabled=false
	pcs property set no-quorum-policy=ignore
	
Create the Virtual IP on the primary server. This VIP will be assigned between the 2 servers. 
If primary goes down, then this ip is assigned to the secondary server. 

	pcs resource create vipbalancerip ocf:heartbeat:IPaddr2 ip=192.168.0.100 cidr_netmask=32 \
                                                                            op monitor interval=30s
	pcs constraint location vipbalancerip prefers primary.server.com=INFINITY

###  Cluster administration

To monitor the status of the cluster:

	pcs status

Here is the output from `primary`

	[root@waepprrkhe001 ~]# pcs status
	Cluster name: vipcluster
	Last updated: Mon Sep 28 20:53:57 2015
	Last change: Mon Sep 28 19:52:47 2015
	Stack: cman
	Current DC: primary.server.com - partition with quorum
	Version: 1.1.11-97629de
	2 Nodes configured
	1 Resources configured
	
	
	Online: [ primary.server.com secondary.server.com ]
	
	Full list of resources:
	
	 livefrontendIP0        (ocf::heartbeat:IPaddr2):       Started primary.server.com
	
	[root@waepprrkhe001 ~]#	


To show the full cluster configuration:

	pcs config

Here is the output from `primary`

	[root@waepprrkhe001 ~]# pcs config
	Cluster Name: vipcluster
	Corosync Nodes:
	 primary.server.com secondary.server.com
	Pacemaker Nodes:
	 primary.server.com secondary.server.com
	
	Resources:
	 Resource: livefrontendIP0 (class=ocf provider=heartbeat type=IPaddr2)
	  Attributes: ip=192.168.0.100 cidr_netmask=32
	  Operations: start interval=0s timeout=20s (livefrontendIP0-start-interval-0s)
	              stop interval=0s timeout=20s (livefrontendIP0-stop-interval-0s)
	              monitor interval=30s (livefrontendIP0-monitor-interval-30s)
	
	Stonith Devices:
	Fencing Levels:
	
	Location Constraints:
	  Resource: livefrontendIP0
	    Enabled on: primary.server.com (score:INFINITY) 
                                        (id:location-livefrontendIP0-primary.server.com-INFINITY)
	Ordering Constraints:
	Colocation Constraints:
	
	Resources Defaults:
	 No defaults set
	Operations Defaults:
	 No defaults set
	
	Cluster Properties:
	 cluster-infrastructure: cman
	 dc-version: 1.1.11-97629de
	 no-quorum-policy: ignore
	 stonith-enabled: false
	[root@waepprrkhe001 ~]#

###  Failover testing.

Shutdown `secondary` server.

	[root@waepprrkhe001 ~]# pcs status
	Cluster name: vipcluster
	Last updated: Mon Sep 28 20:08:00 2015
	Last change: Mon Sep 28 19:52:47 2015
	Stack: cman
	Current DC: primary.server.com - partition WITHOUT quorum
	Version: 1.1.11-97629de
	2 Nodes configured
	1 Resources configured
	
	
	Online: [ primary.server.com ]
	OFFLINE: [ secondary.server.com ]
	
	Full list of resources:
	
	 livefrontendIP0        (ocf::heartbeat:IPaddr2):       Started primary.server.com

Shutdown `primary` server. 

	[root@waepprrkhe002 ~]# pcs status
	Cluster name: vipcluster
	Last updated: Mon Sep 28 20:05:30 2015
	Last change: Mon Sep 28 19:52:47 2015
	Stack: cman
	Current DC: secondary.server.com - partition WITHOUT quorum
	Version: 1.1.11-97629de
	2 Nodes configured
	1 Resources configured
	
	
	Online: [ secondary.server.com ]
	OFFLINE: [ primary.server.com ]
	
	Full list of resources:
	
	 livefrontendIP0        (ocf::heartbeat:IPaddr2):       Started secondary.server.com




