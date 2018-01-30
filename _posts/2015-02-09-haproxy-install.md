---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: How to setup HAProxy
category: ['Linux', 'Ha']
tags: ['linux', 'ha', 'load-balancing', 'high-avaliability', 'proxy', 'nginx', 'haproxy']
---

HAProxy is the Reliable, High Performance TCP/HTTP Load Balancer and it works nicely with Deveo Cluster setup.

Installation on CentOS

### Follow these steps to install on CentOS:

	[ahmed@ahmed-server ~]$ sudo yum install make gcc wget
	[ahmed@ahmed-server ~]$ wget http://www.haproxy.org/download/1.5/src/haproxy-1.5.11.tar.gz
	[ahmed@ahmed-server ~]$ tar -zxvf haproxy-1.5.11.tar.gz -C /opt
	[ahmed@ahmed-server ~]$ cd /opt/haproxy-1.5.11
	[ahmed@ahmed-server haproxy-1.5.11]$ sudo make TARGET=linux26 CPU=x86_64
	[ahmed@ahmed-server haproxy-1.5.11]$ sudo make install

### Follow these steps to create init script:

	[ahmed@ahmed-server ~]$ sudo ln -sf /usr/local/sbin/haproxy /usr/sbin/haproxy
	[ahmed@ahmed-server ~]$ sudo cp /opt/haproxy-1.5.11/examples/haproxy.init /etc/init.d/haproxy
	[ahmed@ahmed-server ~]$ sudo chmod  755 /etc/init.d/haproxy

### Follow these steps to configure haproxy:

	[ahmed@ahmed-server ~]$ sudo mkdir /etc/haproxy
	[ahmed@ahmed-server ~]$ sudo cp /opt/haproxy-1.5.11/examples/examples.cfg \
                                                                /etc/haproxy/haproxy.cfg
	[ahmed@ahmed-server ~]$ sudo mkdir /var/lib/haproxy
	[ahmed@ahmed-server ~]$ sudo touch /var/lib/haproxy/stats
	[ahmed@ahmed-server ~]$ sudo useradd haproxy

### Finally start the service and enable on boot:

	[ahmed@ahmed-server ~]$ sudo service haproxy check
	[ahmed@ahmed-server ~]$ sudo service haproxy start
	[ahmed@ahmed-server ~]$ sudo chkconfig haproxy on

### Configuration sample `haproxy.cfg`.

	global
	        log /dev/log    local0
	        log /dev/log    local1 notice
			log 127.0.0.1	local2
	        # chroot /var/lib/haproxy
	        # stats socket /run/haproxy/admin.sock mode 660 level admin
	        stats timeout 30s
	        user haproxy
	        group haproxy
	        daemon
	
	        #  Default SSL material locations
	        # ca-base /etc/ssl/certs
	        # crt-base /etc/ssl/private
	
	        #  Default ciphers to use on SSL-enabled listening sockets.
	        #  For more information, see ciphers(1SSL).
	        # ssl-default-bind-ciphers 
            #             kEECDH+aRSA+AES:kRSA+AES:+AES256:RC4-SHA:!kEDH:!LOW:!EXP:!MD5:!aNULL:!eNULL
            # 
	defaults
	        log     global
	        mode    http
	        option  httplog
	        option  dontlognull
	        timeout connect 5000
	        timeout client  50000
	        timeout server  50000
	        # errorfile 400 /etc/haproxy/errors/400.http
	        # errorfile 403 /etc/haproxy/errors/403.http
	        # errorfile 408 /etc/haproxy/errors/408.http
	        # errorfile 500 /etc/haproxy/errors/500.http
	        # errorfile 502 /etc/haproxy/errors/502.http
	        # errorfile 503 /etc/haproxy/errors/503.http
	        # errorfile 504 /etc/haproxy/errors/504.http
	
	frontend localnodes
	    bind *:9002
	    mode http
	    default_backend nodes
	
	backend nodes
	    mode http
	    balance roundrobin
	    option forwardfor
	    http-request set-header X-Forwarded-Port %[dst_port]
	    http-request add-header X-Forwarded-Proto https if { ssl_fc }
	    option httpchk HEAD / HTTP/1.1\r\nHost:localhost
	    server web01 127.0.0.1:9090 check
	    server web02 127.0.0.1:9091 check
	    server web03 127.0.0.1:9092 check
	
	listen stats *:9001
	    stats enable
	    stats uri /
	    stats hide-version
	    stats auth someuser:password

###  Configuring Logging

If you look at the top of /etc/haproxy/haproxy.cfg, you will see something like below. If you dont see it then add the line in the beginning.

#### Here is how my conf looks like. 
	
	global
	        log /dev/log    local0
	        log /dev/log    local1 notice
			log 127.0.0.1	local2

If you dont have the below line then add it.
	
	global
		log         127.0.0.1 local2


This means that HAProxy will send its messages to rsyslog on 127.0.0.1. But by default, rsyslog doesn’t listen on any address. 

#### Let’s edit /etc/rsyslog.conf and uncomment these lines:
		
	$ModLoad imudp
	$UDPServerRun 514

This will make rsyslog listen on UDP port 514 for all IP addresses. Optionally you can limit to 127.0.0.1 by adding:

	$UDPServerAddress 127.0.0.1

#### Now create a /etc/rsyslog.d/haproxy.conf file containing:
	
	local2.*    /var/log/haproxy.log

#### You can of course be more specific and create separate log files according to the level of messages:

	
	local2.=info     /var/log/haproxy/haproxy-info.log
	local2.notice    /var/log/haproxy/haproxy-allbutinfo.log

#### Then restart rsyslog and see that log files are created:
	
	#  service rsyslog restart
	Shutting down system logger:                               [  OK  ]
	Starting system logger:                                    [  OK  ]
 
	#  ls -l /var/log/haproxy | grep haproxy
	-rw-------. 1 root   root      131  3 oct.  10:43 haproxy-allbutinfo.log
	-rw-------. 1 root   root      106  3 oct.  10:42 haproxy-info.log

Now you can start your debugging session!

### More Details.

	https://serversforhackers.com/haproxy/
	http://support.deveo.com/knowledgebase/articles/409523-how-to-setup-haproxy
	http://cbonte.github.io/haproxy-dconv/configuration-1.5.html
	http://www.percona.com/blog/2014/10/03/haproxy-give-me-some-logs-on-centos-6-5/