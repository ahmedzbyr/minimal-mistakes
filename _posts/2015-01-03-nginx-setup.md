---
title: Setting up `NGINX` for `HTTP` load balancing.
category: ['Linux']
tags: ['linux', 'nginx', 'http', 'httpd', 'load-balancing', 'high-availability']
---

`NGINX` is a free, open-source, high-performance `HTTP` server and reverse proxy, as well as an IMAP/POP3 proxy server. `NGINX` is known for its high performance, stability, rich feature set, simple configuration, and low resource consumption.

`NGINX` is one of a handful of servers written to address the C10K problem. Unlike traditional servers, `NGINX` doesn’t rely on threads to handle requests. Instead it uses a much more scalable event-driven (asynchronous) architecture. This architecture uses small, but more importantly, predictable amounts of memory under load. Even if you don’t expect to handle thousands of simultaneous requests, you can still benefit from `NGINX`’s high-performance and small memory footprint. `NGINX` scales in all directions: from the smallest VPS all the way up to large clusters of servers.


Below are the 3 server which run a web application which needs to be load balanced.

	nginx-server : nginx.server.com
	Server1 : node1.application.com 
	Server2 : node2.application.com 
	Server3 : node3.application.com 

Install `NGINX` on RHEL/Centos6

#### Step 1 : Create a Repo. in [vim /etc/yum.repos.d/nginx.repo]

	CENTOS6
	[nginx]
	name=nginx repo
	baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
	gpgcheck=0
	enabled=1

	RHEL:
	[nginx]
	name=nginx repo
	baseurl=http://nginx.org/packages/rhel/$releasever/$basearch/
	gpgcheck=0
	enabled=1
	
	OR 
	
#### Step 1a: Or we can install the epel release-6-8.

	[root@httpd-primary ~]# wget \
                http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm

Change the 'https' to 'http' in 'epel.repo', if you get an error when you do a 'yum install heartbeat'

	[root@httpd-primary ~]# vim /etc/yum.repos.d/epel.repo
	[root@httpd-primary ~]# yum install heartbeat	

#### Step 2 : Install 

	ahmed@ngnix ~]$ sudo yum install nginx

#### Step 3 : Configuration:

	#------BEGIN CONFIG------------
	user  nginx;
	worker_processes  1;
	error_log  /var/log/nginx/error.log warn;
	pid        /var/run/nginx.pid;

	events {
		worker_connections  1024;
	}

	http {
		include       /etc/nginx/mime.types;
		default_type  application/octet-stream;

		log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
						  '$status $body_bytes_sent "$http_referer" '
						  '"$http_user_agent" "$http_x_forwarded_for"';

		access_log  /var/log/nginx/access.log  main;

		sendfile        on;
		# tcp_nopush     on;
		keepalive_timeout  65;
		# gzip  on;
		# include /etc/nginx/conf.d/*.conf;
		
		upstream myappredirect 
		{
			server node1.zabbix.com;
			server node2.zabbix.com;
		}
		server 
		{
		listen 80;
			location / {
				proxy_pass http://myappredirect;
			}    
		}
	}
	#------END CONFIG-----------


Now we are ready for testing.

	[ahmed@ngnix ~]$ sudo service nginx start

Now we can go the browser and hit the IP address of nginx server.
As per the current setup (Default) it will do a Round-Robin to send request to the servers.
"All inbound request to `NGINX`[nginx.server.com]" -> `NGINX` will load_balance and distribute traffic to Servers.
 
	-->[`NGINX`]---->[HOST1]
				\-->[HOST2]
				\-->[HOST3]

				
Useful Links:

[Nginx Overview](http://nginx.com/resources/admin-guide/load-balancer/#overview)

[Nginx LB](http://nginx.org/en/docs/http/load_balancing.html)

[Nginx Install](http://wiki.nginx.org/Install)