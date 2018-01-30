---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: Creating Virtual Interface and Assign Multiple IP Addresses.	
category: ['Linux']
tags: ['linux', 'vip', 'network', 'multiple-ip']
---

Current IP of the server : `10.130.18.11`. Virtual IPs being assigned to server : `10.130.18.22`, `10.130.18.23`, `10.130.18.24` to our server.. 
	
Go to `network-scripts` directory and copy the existing `ifcfg-eth0` file.
Make sure you are using a static ip for your server.  	

	cd /etc/sysconfig/network-scripts/

`static` ip assigned scripts looks as below.
	
	[ahmed@ahmed-server network-scripts]$ sudo vim ifcfg-eth0

	DEVICE="eth0"
	BOOTPROTO=static
	NM_CONTROLLED="no"
	ONBOOT=yes
	TYPE="Ethernet"
	IPADDR=10.130.18.11
	NETMASK=255.255.255.192
	GATEWAY=10.138.18.1
	HWADDR=A0:0C:29:28:A7:4C 

Make sure we copy the same script as ifcfg-eth0:0/1/2 	
		
	sudo cp ifcfg-eth0 ifcfg-eth0:0
	sudo cp ifcfg-eth0:0 ifcfg-eth0:1
	sudo cp ifcfg-eth0:0 ifcfg-eth0:2

Change the copied script as below.
Here we are assigned the Virtual IP addresses to the Server.	

####`ifcfg-eth0:0` Configuration
	
	[ahmed@ahmed-server network-scripts]$ sudo vim ifcfg-eth0:0

	DEVICE="eth0:0"
	BOOTPROTO=static
	NM_CONTROLLED="no"
	ONBOOT=yes
	TYPE="Ethernet"
	IPADDR=10.130.18.22
	NETMASK=255.255.255.192
	GATEWAY=10.138.18.1
	HWADDR=A0:0C:29:28:A7:4C 

####`ifcfg-eth0:1` Configuration
 	
	[ahmed@ahmed-server network-scripts]$ sudo vim ifcfg-eth0:1

	DEVICE="eth0:1"
	BOOTPROTO=static
	NM_CONTROLLED="no"
	ONBOOT=yes
	TYPE="Ethernet"
	IPADDR=10.130.18.23
	NETMASK=255.255.255.192
	GATEWAY=10.138.18.1
	HWADDR=A0:0C:29:28:A7:4C 
	 
####`ifcfg-eth0:2` Configuration
	 
	[ahmed@ahmed-server network-scripts]$ sudo vim ifcfg-eth0:2

	DEVICE="eth0:2"
	BOOTPROTO=static
	NM_CONTROLLED="no"
	ONBOOT=yes
	TYPE="Ethernet"
	IPADDR=10.130.18.24
	NETMASK=255.255.255.192
	GATEWAY=10.138.18.1
	HWADDR=A0:0C:29:28:A7:4C  

Now lets restart the network so that the changes take affect. 
	
	[ahmed@ahmed-server network-scripts]$ sudo /etc/init.d/network restart
	Shutting down interface eth0:                              [  OK  ]
	Shutting down loopback interface:                          [  OK  ]
	Bringing up loopback interface:                            [  OK  ]
	Bringing up interface eth0:  Determining if ip address 10.130.18.11 is already 
                                                                            in use for device eth0...
															   [  OK  ]


Checking the configuration.															   
															   
	[ahmed@ahmed-server network-scripts]$ ifconfig
	eth0      Link encap:Ethernet  HWaddr A0:0C:29:28:A7:4C
			  inet addr:10.130.18.11  Bcast:10.130.18.63  Mask:255.255.255.192
			  inet6 addr: fe80::a2d3:c1ff:fef9:d8dc/64 Scope:Link
			  UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
			  RX packets:74 errors:0 dropped:0 overruns:0 frame:0
			  TX packets:56 errors:0 dropped:0 overruns:0 carrier:0
			  collisions:0 txqueuelen:1000
			  RX bytes:6687 (6.5 KiB)  TX bytes:10366 (10.1 KiB)
			  Interrupt:32

	eth0:0    Link encap:Ethernet  HWaddr A0:0C:29:28:A7:4C
			  inet addr:10.130.18.22  Bcast:10.130.18.63  Mask:255.255.255.192
			  UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
			  Interrupt:32

	eth0:1    Link encap:Ethernet  HWaddr A0:0C:29:28:A7:4C
			  inet addr:10.130.18.23  Bcast:10.130.18.63  Mask:255.255.255.192
			  UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
			  Interrupt:32

	eth0:2    Link encap:Ethernet  HWaddr A0:0C:29:28:A7:4C
			  inet addr:10.130.18.24  Bcast:10.130.18.63  Mask:255.255.255.192
			  UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
			  Interrupt:32
	
Let Ping those IPs.

	ahmed@ahmed-second-server:~# ping 10.130.18.11
	PING 10.130.18.11 (10.130.18.11) 56(84) bytes of data.
	64 bytes from 10.130.18.11: icmp_req=2 ttl=59 time=0.288 ms
	64 bytes from 10.130.18.11: icmp_req=3 ttl=59 time=0.962 ms
	64 bytes from 10.130.18.11: icmp_req=4 ttl=59 time=0.287 ms
	^C
	--- 10.130.18.11 ping statistics ---
	4 packets transmitted, 3 received, 25% packet loss, time 3008ms
	rtt min/avg/max/mdev = 0.287/0.512/0.962/0.318 ms
	ahmed@ahmed-second-server:~# ping 10.130.18.22
	PING 10.130.18.22 (10.130.18.22) 56(84) bytes of data.
	64 bytes from 10.130.18.22: icmp_req=1 ttl=59 time=0.680 ms
	64 bytes from 10.130.18.22: icmp_req=2 ttl=59 time=1.67 ms
	64 bytes from 10.130.18.22: icmp_req=3 ttl=59 time=0.274 ms
	^C
	--- 10.130.18.22 ping statistics ---
	3 packets transmitted, 3 received, 0% packet loss, time 2004ms
	rtt min/avg/max/mdev = 0.274/0.877/1.678/0.590 ms
	ahmed@ahmed-second-server:~# ping 10.130.18.23
	PING 10.130.18.23 (10.130.18.23) 56(84) bytes of data.
	64 bytes from 10.130.18.23: icmp_req=2 ttl=59 time=0.853 ms
	64 bytes from 10.130.18.23: icmp_req=3 ttl=59 time=0.626 ms
	64 bytes from 10.130.18.23: icmp_req=4 ttl=59 time=0.346 ms
	^C
	--- 10.130.18.23 ping statistics ---
	4 packets transmitted, 3 received, 25% packet loss, time 3014ms
	rtt min/avg/max/mdev = 0.346/0.608/0.853/0.208 ms 


## Assign Multiple IP Address Range

If you would like to create a range of Multiple IP Addresses to a particular interface called “ifcfg-eth0“, we use “ifcfg-eth0-range0” and copy the contains of ifcfg-eth0 on it as shown below.

	[ahmed@ahmed-server network-scripts]$ sudo cp -p ifcfg-eth0 ifcfg-eth0-range0

Now open “ifcfg-eth0-range0” file and add “IPADDR_START” and “IPADDR_END” IP address range as shown below.

	[ahmed@ahmed-server network-scripts]# vi ifcfg-eth0-range0
	
	# DEVICE="eth0"
	# BOOTPROTO=static
	# NM_CONTROLLED="no"
	# ONBOOT=yes
	TYPE="Ethernet"
	IPADDR_START=10.130.18.22
	IPADDR_END=10.130.18.24
	IPV6INIT=no

Save it and restart/start network service

	[ahmed@ahmed-server network-scripts]$ sudo /etc/init.d/network restart
	Shutting down interface eth0:                              [  OK  ]
	Shutting down loopback interface:                          [  OK  ]
	Bringing up loopback interface:                            [  OK  ]
	Bringing up interface eth0:  Determining if ip address 10.130.18.11 is already in 
                                                                            use for device eth0...
															   [  OK  ]


Checking the configuration.	Verify that virtual interfaces are created with IP Address.														   
															   
	[ahmed@ahmed-server network-scripts]$ ifconfig
	eth0      Link encap:Ethernet  HWaddr A0:0C:29:28:A7:4C
			  inet addr:10.130.18.11  Bcast:10.130.18.63  Mask:255.255.255.192
			  inet6 addr: fe80::a2d3:c1ff:fef9:d8dc/64 Scope:Link
			  UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
			  RX packets:74 errors:0 dropped:0 overruns:0 frame:0
			  TX packets:56 errors:0 dropped:0 overruns:0 carrier:0
			  collisions:0 txqueuelen:1000
			  RX bytes:6687 (6.5 KiB)  TX bytes:10366 (10.1 KiB)
			  Interrupt:32

	eth0:0    Link encap:Ethernet  HWaddr A0:0C:29:28:A7:4C
			  inet addr:10.130.18.22  Bcast:10.130.18.63  Mask:255.255.255.192
			  UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
			  Interrupt:32

	eth0:1    Link encap:Ethernet  HWaddr A0:0C:29:28:A7:4C
			  inet addr:10.130.18.23  Bcast:10.130.18.63  Mask:255.255.255.192
			  UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
			  Interrupt:32

	eth0:2    Link encap:Ethernet  HWaddr A0:0C:29:28:A7:4C
			  inet addr:10.130.18.24  Bcast:10.130.18.63  Mask:255.255.255.192
			  UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
			  Interrupt:32
	 
More Info : 

	http://www.tecmint.com/create-multiple-ip-addresses-to-one-single-network-interface/
	http://linuxconfig.org/configuring-virtual-network-interfaces-in-linux
	http://www.jamescoyle.net/how-to/307-create-a-virtual-ip-address-in-linux
	https://myunixlab.wordpress.com/2011/02/04/how-to-add-virtual-ip-address-in-linux/