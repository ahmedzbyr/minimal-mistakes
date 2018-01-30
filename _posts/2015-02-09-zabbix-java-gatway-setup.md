---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: Installing `zabbix-java-gateway` on Centos 6.5
category: ['Linux', 'Zabbix', 'Nagios', 'Monitoring']
tags: ['zabbix', 'linux', 'java', 'zabbix-java-gateway', 'centos', 'rhel', 'nagios', 'monitoring']
---

Zabbix 2.0 adds native support for monitoring JMX applications by introducing a new Zabbix daemon called `Zabbix Java gateway`. `Zabbix Java gateway` is a daemon written in Java. When Zabbix server wants to know the value of a particular JMX counter on a host, it asks `Zabbix Java gateway`, which uses the `JMX` management API to query the application of interest remotely. The application does not need any additional software installed, it just has to be started with `-Dcom.sun.management.jmxremote` option on the command line.

###  How does `zabbix-java-gateway` work?
	
>1. First we configure system which needs to be monitored using the `JAVA_OPTS`.
>2. Next we add a `JMX Interface` in Zabbix server UI under `hosts`.
>3. `zabbix-server` will communicate with `zabbix-java-gateway` which intern communicates to the system/server where we need to get all the JMX data.
>4. JMX is set using the `JAVA_OPTS`.  

	[Zabbix-Server] 
                |
                +--(port:10053)--> 
                               [zabbix-java-gateway] 
                                          |
                                          +--(port:12345)--> 
                                                    [JMX enabled server, Example:Tomcat/WebServer]

###  Step 1 : Install `zabbix-java-gateway` on `zabbix-server`
	
	[ahmed@ahmed-server ~]$ sudo yum install zabbix-java-gateway


###  Step 2 : Configure the `host` with `JMX` which needs to be monitored. 

Setting Tomcat/JMX Options. Add the below lines to `setenv.sh` and save it under `apache-tomcat-7/bin/`
So when the `start.sh` is started then these `JMX` options will be added to `tomcat` server. 

**NOTE: To make the monitoring secure use ssl and authentication options. You can find more information in the links at the end of this post.**

IMPORTANT Lines are below. We will be getting data from port `12345`. 

	 -Dcom.sun.management.jmxremote\
	 -Dcom.sun.management.jmxremote.port=12345\
	 -Dcom.sun.management.jmxremote.authenticate=false\
	 -Dcom.sun.management.jmxremote.ssl=false"

But here are the complete `JAVA_OPTS`. you can ignore the first few lines which sets the Heap Memory size.

	export JAVA_OPTS="$JAVA_OPTS\
	 -server\
	 -Xms1024m\
	 -Xmx2048m\
	 -XX:MaxPermSize=256m\
	 -XX:MaxNewSize=256m\
	 -XX:NewSize=256m\
	 -XX:SurvivorRatio=12\
	 -Dcom.sun.management.jmxremote\
	 -Dcom.sun.management.jmxremote.port=12345\
	 -Dcom.sun.management.jmxremote.authenticate=false\
	 -Dcom.sun.management.jmxremote.ssl=false"

###  Step 3 : Configuring `zabbix-server`. 

>1. Here we configure the zabbix server to let it know where the `zabbix-java-gateway` is running. 
>2. Since we are running the `zabbix-java-gateway` in the same server as `zabbix-server`, so we will be using the same `ip` for both. 
>3. Only difference is that `zabbix-java-gateway` will be running on port `10053`

Configuration in `zabbix-server.conf`. Add the below line. Rather un-comment them and add the IP/ports

	###  Option: JavaGateway
	#        IP address (or hostname) of Zabbix Java gateway.
	#        Only required if Java pollers are started.
	# 
	#  Mandatory: no
	#  Default:
	JavaGateway=10.10.18.27
	
	###  Option: JavaGatewayPort
	#        Port that Zabbix Java gateway listens on.
	# 
	#  Mandatory: no
	#  Range: 1024-32767
	#  Default:
	JavaGatewayPort=10053
	
	###  Option: StartJavaPollers
	#        Number of pre-forked instances of Java pollers.
	# 
	#  Mandatory: no
	#  Range: 0-1000
	#  Default:
	StartJavaPollers=5


###  Step 4 : Configuring `zabbix-java-gateway`. 

>1. We now set where the `zabbix-java-gateway` will be running and which port it will be listing on.
>2. Configuration in `zabbix-java-gateway`, here same `ip` as the `zabbix-server` and port `10053`. 


	###  Option: zabbix.listenIP
	#        IP address to listen on.
	# 
	#  Mandatory: no
	#  Default:
	LISTEN_IP="10.10.18.27"
	
	###  Option: zabbix.listenPort
	#        Port to listen on.
	# 
	#  Mandatory: no
	#  Range: 1024-32767
	#  Default:
	LISTEN_PORT=10053
	
	###  Option: zabbix.pidFile
	#        Name of PID file.
	#        If omitted, Zabbix Java Gateway is started as a console application.
	# 
	#  Mandatory: no
	#  Default:
	#  PID_FILE=
	
	PID_FILE="/var/run/zabbix/zabbix_java.pid"
	
	###  Option: zabbix.startPollers
	#        Number of worker threads to start.
	# 
	#  Mandatory: no
	#  Range: 1-1000
	#  Default:
	START_POLLERS=5


### Useful Links:

	https://www.zabbix.com/documentation/2.4/manual/concepts/java
	https://www.zabbix.com/documentation/2.4/manual/config/items/itemtypes/jmx_monitoring
