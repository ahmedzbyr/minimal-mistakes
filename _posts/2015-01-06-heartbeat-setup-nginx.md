---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: HA Setup - `heartbeat` for `nginx`.
category: ['Linux']
tags: ['linux', 'heartbeat', 'nginx', 'http', 'httpd', 'replication', 'load-balancing', 'high-availability']
---

`Heartbeat` is a daemon that provides cluster infrastructure (communication and membership) services to its clients. This allows clients to know about the presence (or disappearance!) of peer processes on other machines and to easily exchange messages with them. `NGINX` is a free, open-source, high-performance HTTP server and reverse proxy, as well as an IMAP/POP3 proxy server. NGINX is known for its high performance, stability, rich feature set, simple configuration, and low resource consumption.

####  Step 1: Pre-Configuration Requirements

Assign hostname nginx-primary to primary node with IP address 192.168.230.138 to eth0.
Assign hostname nginx-slave to slave node with IP address 192.168.230.139 to eth0.

	[root@nginx-primary ~]# cat /etc/hosts
	127.0.0.1   localhost
	192.168.230.138         nginx-primary
	192.168.230.139         nginx-slave

Primary Server: nginx-primary

	[root@nginx-primary ~]# uname -n
	nginx-primary

Secondary Server: nginx-slave

	[root@nginx-slave ~]# uname -n
	nginx-slave

192.168.230.150 is the virtual IP address that will be used for our nginx webserver.
Configure nginx on both server is 'nginx-primary' and 'nginx-slave'

Now lets install Heartbeat on both servers.

####  Step 2: Install EPEL repo.

	[root@nginx-primary ~]# wget \
                http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm

Change the 'https' to 'http' in 'epel.repo', if you get an error when you do a 'yum install heartbeat'

	[root@nginx-primary ~]# vim /etc/yum.repos.d/epel.repo
	[root@nginx-primary ~]# yum install heartbeat

####  Step 3: Heartbeat configuration

	[root@nginx-primary ~]# cp /usr/share/doc/heartbeat-3.0.4/authkeys /etc/ha.d/
	[root@nginx-primary ~]# cp /usr/share/doc/heartbeat-3.0.4/ha.cf /etc/ha.d/
	[root@nginx-primary ~]# cp /usr/share/doc/heartbeat-3.0.4/haresources /etc/ha.d/

####  Step 4: 

	[root@nginx-primary ~]# vi /etc/ha.d/authkeys

Then add the following lines:

	auth 2
	2 sha1 anyRandomKeyHere


####  Step 5: Change the permission of the authkeys file:

	chmod 600 /etc/ha.d/authkeys

####  Step 6: Moving to our second file (ha.cf) which is the most important. So edit the ha.cf file with vi:

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
	node nginx-primary
	node nginx-slave

####  Step 7: update haresources

	vi /etc/ha.d/haresources

Add the following line: 

	nginx-primary 192.168.230.150 nginx

####  Step 8: Copy the /etc/ha.d/ directory from nginx-primary to nginx-slave:

	scp -r /etc/ha.d/ root@nginx-slave:/etc/

####  Step 9: As we want nginx highly enabled let's start configuring nginx:

	vi /etc/nginx/nginx.conf

Add this line in nginx.conf:

	Listen 192.168.230.150:80

####  Step 10: Copy the /etc/nginx/conf/nginx.conf file from nginx-primary to nginx-slave:

	scp /etc/nginx/nginx.conf root@nginx-slave:/etc/nginx/

####  Step 11. Now start heartbeat on the primary nginx-primary and slave nginx-slave:

	/etc/init.d/heartbeat start

####  Step 12. Open web-browser and type in the URL:

	http://192.168.230.150
	
####  Step 13. Now stop the hearbeat daemon on nginx-primary:

	/etc/init.d/heartbeat stop

In your browser type in the URL 

	http://192.168.230.150 and press enter.


*We don't need to create a virtual network interface and assign an IP address (192.168.230.150) to it.* 
*Heartbeat will do this for you, and start the service (nginx) itself. So don't worry about this.*

#####NOTE : 
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

	https://wiki.archlinux.org/index.php/Simple_IP_Failover_with_Heartbeat
	http://www.linuxnix.com/2010/01/heartbeat-clustering.html
	https://www.howtoforge.com/high_availability_heartbeat_centos
	http://blog.iwayvietnam.com/tuanta/2010/05/19/configuring-a-high-availability-cluster-on-rhel-centos/
	http://www.howtogeek.com/177621/the-beginners-guide-to-iptables-the-linux-firewall/
	http://www.cyberciti.biz/tips/linux-cluster-building-firewall-rules.html
	http://www.linux-ha.org/doc/users-guide/_creating_an_initial_heartbeat_configuration.html
