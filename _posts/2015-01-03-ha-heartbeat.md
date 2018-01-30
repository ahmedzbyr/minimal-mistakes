---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: httpd HA using Heartbeat.
category: ['Linux']
tags: ['linux', 'heartbeat', 'high-availability', 'httpd', 'http']
---

`httpd` is the Apache HyperText Transfer Protocol (HTTP) server program. It is designed to be run as a standalone daemon process. When used like this it will create a pool of child processes or threads to handle requests. `Heartbeat` is a daemon that provides cluster infrastructure (communication and membership) services to its clients. This allows clients to know about the presence (or disappearance!) of peer processes on other machines and to easily exchange messages with them.

#### Step 1: Pre-Configuration Requirements

Assign hostname httpd-primary to primary node with IP address 192.168.230.138 to eth0.
Assign hostname httpd-slave to slave node with IP address 192.168.230.139 to eth0.

	[root@httpd-primary ~]# cat /etc/hosts
	127.0.0.1   localhost
	192.168.230.138         httpd-primary
	192.168.230.139         httpd-slave

Primary Server: httpd-primary

	[root@httpd-primary ~]# uname -n
	httpd-primary

Secondary Server: httpd-slave

	[root@httpd-slave ~]# uname -n
	httpd-slave

192.168.230.150 is the virtual IP address that will be used for our httpd webserver.
Configure httpd on both server is 'httpd-primary' and 'httpd-slave'

Now lets install Heartbeat on both servers.

#### Step 2: Install EPEL repo.

	[root@httpd-primary ~]# wget \
                http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm

Change the 'https' to 'http' in 'epel.repo', if you get an error when you do a 'yum install heartbeat'

	[root@httpd-primary ~]# vim /etc/yum.repos.d/epel.repo
	[root@httpd-primary ~]# yum install heartbeat

#### Step 3: Heartbeat configuration

	[root@httpd-primary ~]# cp /usr/share/doc/heartbeat-3.0.4/authkeys /etc/ha.d/
	[root@httpd-primary ~]# cp /usr/share/doc/heartbeat-3.0.4/ha.cf /etc/ha.d/
	[root@httpd-primary ~]# cp /usr/share/doc/heartbeat-3.0.4/haresources /etc/ha.d/

#### Step 4:

	[root@httpd-primary ~]# vi /etc/ha.d/authkeys

Then add the following lines:

	auth 2
	2 sha1 anyRandomKeyHere


#### Step 5: Change the permission of the authkeys file:

	chmod 600 /etc/ha.d/authkeys

#### Step 6: Moving to our second file (ha.cf) which is the most important. So edit the ha.cf file with vi:

	vi /etc/ha.d/ha.cf

Add the following lines in the ha.cf file:

	logfile /var/log/ha-log
	logfacility local0
	keepalive 2
	deadtime 30
	initdead 120
	bcast eth0
	udpport 694
	auto_failback on
	node httpd-primary
	node httpd-slave

#### Step 7: update haresources

	vi /etc/ha.d/haresources

Add the following line:

	httpd-primary 192.168.230.150 httpd

#### Step 8: Copy the /etc/ha.d/ directory from httpd-primary to httpd-slave:

	scp -r /etc/ha.d/ root@httpd-slave:/etc/

#### Step 9: As we want httpd highly enabled let's start configuring httpd:

	vi /etc/httpd/conf/httpd.conf

Add this line in httpd.conf:

	Listen 192.168.230.150:80

#### Step 10: Copy the /etc/httpd/conf/httpd.conf file from httpd-primary to httpd-slave:

	scp /etc/httpd/conf/httpd.conf root@httpd-slave:/etc/httpd/conf/

#### Step 11: Create the file index.html on both nodes (httpd-primary & httpd-slave):
On httpd-primary:

	echo "`uname -n` apache test server" > /var/www/html/index.html

On httpd-slave:

	echo "`uname -n` apache test server" > /var/www/html/index.html

#### Step 11. Now start heartbeat on the primary httpd-primary and slave httpd-slave:

	/etc/init.d/heartbeat start

#### Step 12. Open web-browser and type in the URL:

	http://192.168.230.150
	It will show 'httpd-primary apache test server'.

#### Step 13. Now stop the hearbeat daemon on httpd-primary:

	/etc/init.d/heartbeat stop

In your browser type in the URL

	http://192.168.230.150 and press enter.
	It will show 'httpd-slave apache test server'.

*We don't need to create a virtual network interface and assign an IP address (192.168.230.150) to it.*
*Heartbeat will do this for you, and start the service (httpd) itself. So don't worry about this.*

#### NOTE :
A. I had issues due to port access for testing disable 'iptables' if you face any issues.
Here are the entries requires for iptables.

	iptables -A OUTPUT -o lo -p udp -m udp -d 192.168.230.138 --dport 694 -j ACCEPT
	iptables -A INPUT -i lo -p udp -m udp -d 192.168.230.138 --dport 694 -j ACCEPT
	iptables -A OUTPUT -o eth0 -p udp -m udp -d 192.168.230.138 --dport 694 -j ACCEPT
	iptables -A INPUT -i eth0 -p udp -m udp -d 192.168.230.138 --dport 694 -j ACCEPT

	iptables -A OUTPUT -o lo -p udp -m udp -d 192.168.230.139 --dport 694 -j ACCEPT
	iptables -A INPUT -i lo -p udp -m udp -d 192.168.230.139 --dport 694 -j ACCEPT
	iptables -A OUTPUT -o eth0 -p udp -m udp -d 192.168.230.139 --dport 694 -j ACCEPT
	iptables -A INPUT -i eth0 -p udp -m udp -d 192.168.230.139 --dport 694 -j ACCEPT

	iptables -A OUTPUT -o eth0 -p tcp -m tcp -d 192.168.230.138 --dport 80 -j ACCEPT
	iptables -A INPUT -i eth0 -p tcp -m tcp -d 192.168.230.138 --dport 80 -j ACCEPT

	iptables -A OUTPUT -o eth0 -p tcp -m tcp -d 192.168.230.139 --dport 80 -j ACCEPT
	iptables -A INPUT -i eth0 -p tcp -m tcp -d 192.168.230.139 --dport 80 -j ACCEPT

	service iptables save


Useful Links:

[Heartbeat](https://wiki.archlinux.org/index.php/Simple_IP_Failover_with_Heartbeat)

[Heartbeat Clustering](http://www.linuxnix.com/2010/01/heartbeat-clustering.html)

[How To Forge](https://www.howtoforge.com/high_availability_heartbeat_centos)

[Blog Tuanta](http://blog.iwayvietnam.com/tuanta/2010/05/19/configuring-a-high-availability-cluster-on-rhel-centos/)

[How to Geek](http://www.howtogeek.com/177621/the-beginners-guide-to-iptables-the-linux-firewall/)

[Cyber Citi](http://www.cyberciti.biz/tips/linux-cluster-building-firewall-rules.html)

[Linux HA](http://www.linux-ha.org/doc/users-guide/_creating_an_initial_heartbeat_configuration.html)
