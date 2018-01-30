---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: Interface Forwarding - from `eth1` to `eth0` on `EDGE` node.
category: ['Hadoop', 'Linux']
tags: ['linux', 'hadoop', 'interface', 'network']
---

Adding `route` to all the `slaves` which reside on a private network to communicate with `External Server` directly using an `EDGE` node using `Interface Forwarding`.

NOTE : Below testing was done on RHEL 6.6

What we are trying to do.

1. All the slave nodes will send their data to Edge nodes on a private interface.
2. Edge Node will take the data arriving on the `private` interface and forward it over a external interface.  

NOTE: below I have used `slave`s for all the nodes which are communicating with `EDGE`, in the this case making `EDGE` as the `master` which acts like a `router`.

###  Datanodes `ifconfig`

Slaves will only run on Private network.

1. `192.168.0.11` aka `eth1` Private Interface.

Here is the `ifconfig`.

	[root@slave-node ~]# ifconfig
	eth0      Link encap:Ethernet  HWaddr 
	          inet addr:192.168.0.8  Bcast:192.168.0.255  Mask:255.255.255.0
	          inet6 addr: fe80::21d:d8ff:feb7:1efe/64 Scope:Link
	          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
	          RX packets:131581 errors:0 dropped:0 overruns:0 frame:0
	          TX packets:148636 errors:0 dropped:0 overruns:0 carrier:0
	          collisions:0 txqueuelen:1000
	          RX bytes:11583580 (11.0 MiB)  TX bytes:35866144 (34.2 MiB)
	
	lo        Link encap:Local Loopback
	          inet addr:127.0.0.1  Mask:255.0.0.0
	          inet6 addr: ::1/128 Scope:Host
	          UP LOOPBACK RUNNING  MTU:65536  Metric:1
	          RX packets:245626 errors:0 dropped:0 overruns:0 frame:0
	          TX packets:245626 errors:0 dropped:0 overruns:0 carrier:0
	          collisions:0 txqueuelen:0
	          RX bytes:286415155 (273.1 MiB)  TX bytes:286415155 (273.1 MiB)

Edge Node `ifconfig`

1. `172.14.14.214` aka `eth0` External Interface
2. `192.168.0.11` aka `eth1` Private Interface.

Here is the `ifconfig`.

	[root@edge-node ~]# ifconfig
	eth0      Link encap:Ethernet  HWaddr 
	          inet addr:172.14.14.214  Bcast:172.14.14.255  Mask:255.255.255.0
	          inet6 addr: fe80::21d:d8ff:feb7:1f7b/64 Scope:Link
	          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
	          RX packets:908442 errors:0 dropped:0 overruns:0 frame:0
	          TX packets:235173 errors:0 dropped:0 overruns:0 carrier:0
	          collisions:0 txqueuelen:1000
	          RX bytes:77363514 (73.7 MiB)  TX bytes:33167098 (31.6 MiB)
	
	eth1      Link encap:Ethernet  HWaddr 
	          inet addr:192.168.0.11  Bcast:192.168.0.255  Mask:255.255.255.0
	          inet6 addr: fe80::21d:d8ff:feb7:1f7a/64 Scope:Link
	          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
	          RX packets:210510 errors:0 dropped:0 overruns:0 frame:0
	          TX packets:177170 errors:0 dropped:0 overruns:0 carrier:0
	          collisions:0 txqueuelen:1000
	          RX bytes:61583138 (58.7 MiB)  TX bytes:16125613 (15.3 MiB)
	
	lo        Link encap:Local Loopback
	          inet addr:127.0.0.1  Mask:255.0.0.0
	          inet6 addr: ::1/128 Scope:Host
	          UP LOOPBACK RUNNING  MTU:65536  Metric:1
	          RX packets:13799253 errors:0 dropped:0 overruns:0 frame:0
	          TX packets:13799253 errors:0 dropped:0 overruns:0 carrier:0
	          collisions:0 txqueuelen:0
	          RX bytes:27734863794 (25.8 GiB)  TX bytes:27734863794 (25.8 GiB)

	[root@edge-node ~]#

###  Configuration.


1. Create `FORWARD`er on the Edge node.
2. Create `route` on all the `slave`.
3. Update `/etc/hosts` on `slave` nodes.


####  1. Create `FORWARD`er on the `Edge` node.
 
1. If you haven't already enabled forwarding in the kernel, do so.
2. Open `/etc/sysctl.conf` and uncomment `net.ipv4.ip_forward = 1`
3. Then execute `$ sudo sysctl -p`
4. Add the following rules to `iptables`

Commands.

    [root@edge-node ~]# iptables -t nat -A POSTROUTING --out-interface eth0 -j MASQUERADE  
    [root@edge-node ~]# iptables -A FORWARD --in-interface eth1 -j ACCEPT


####  2. Create `route` on all the `slave`.

Here is the command to add the route in `slaves`.

	[root@datanode ~]# route add -net 172.0.0.0 netmask 255.0.0.0 gw 192.168.0.11 eth0

We are tell all the traffic trying to go to `172.x.x.x` will have to use `192.168.0.11` as the gateway. Which is the `Private` Interface on the `Edge` Node. 


####  3. Update `/etc/hosts` on `slave` nodes.

And then we update the `/etc/hosts` file with the direct `IP` of External Server `172.14.14.174`, as `slave` node now should be able to communicate to the External Server.

	[root@slave-nodes ~]# ping  172.14.14.174
	PING 172.14.14.174 (172.14.14.174) 56(84) bytes of data.
	64 bytes from 172.14.14.174: icmp_seq=1 ttl=127 time=1.02 ms
	64 bytes from 172.14.14.174: icmp_seq=2 ttl=127 time=1.04 ms
	64 bytes from 172.14.14.174: icmp_seq=3 ttl=127 time=1.03 ms
	^C
	--- 172.14.14.174 ping statistics ---
	3 packets transmitted, 3 received, 0% packet loss, time 2892ms
	rtt min/avg/max/mdev = 1.020/1.033/1.049/0.038 ms
	[root@slave-nodes ~]# ping  172.14.14.141
	PING 172.14.14.141 (172.14.14.141) 56(84) bytes of data.
	64 bytes from 172.14.14.141: icmp_seq=1 ttl=127 time=0.968 ms
	64 bytes from 172.14.14.141: icmp_seq=2 ttl=127 time=1.01 ms
	64 bytes from 172.14.14.141: icmp_seq=3 ttl=127 time=3.73 ms
	^C
	--- 172.14.14.141 ping statistics ---
	3 packets transmitted, 3 received, 0% packet loss, time 2350ms
	rtt min/avg/max/mdev = 0.968/1.906/3.732/1.291 ms

