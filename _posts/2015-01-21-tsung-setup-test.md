---
title: Installing and Initial setup of Tsung Load Testing CentOS
category: ['Linux', 'Testing']
tags: ['load-testing', 'performance', 'seige', 'tsung', 'testing']
---

Tsung (formerly known as `idx-Tsunami`) is a stress testing tool written in the Erlang language and distributed under the GPL license. It can currently stress test `HTTP`, `WebDAV`, `LDAP`, `MySQL`, `PostgreSQL`, `SOAP` and `XMPP` servers. Tsung can simulate hundreds of simultaneous users on a single system. It can also function in a clustered environment.

### Installation on Centos

	[ahmed@server ~]$ yum install erlang 
	[ahmed@server ~]$ tar -xvzf v1.5.1.tar.gz -C /opt
	[ahmed@server ~]$ cd /opt/tsung-1.5.1
	[ahmed@server ~]$ ./configure
	[ahmed@server ~]$ make
	[ahmed@server ~]$ make install
	
Some Version information.

	[ahmed@server ~]$ tsung -v
	Tsung version 1.5.1
	[ahmed@server ~]$ tsung
	Usage: tsung <options> start|stop|debug|status
	Options:
		-f <file>     set configuration file (default is ~/.tsung/tsung.xml)
					   (use - for standard input)
		-l <logdir>   set log directory where YYYYMMDD-HHMM dirs are created 
                            (default is ~/.tsung/log/)
		-i <id>       set controller id (default is empty)
		-r <command>  set remote connector (default is ssh)
		-s            enable erlang smp on client nodes
		-p <max>      set maximum erlang processes per vm (default is 250000)
		-m <file>     write monitoring output on this file (default is tsung.log)
					   (use - for standard output)
		-F            use long names (FQDN) for erlang nodes
		-w            warmup delay (default is 1 sec)
		-v            print version information and exit
		-6            use IPv6 for Tsung internal communications
		-x <tags>     list of requests tag to be excluded from the run (separated by comma)
		-h            display this help and exit
	[ahmed@server ~]$


Sample Test `load.xml` update `load_test_machine` and `web_server_to_test` as per the servers.

	<?xml version="1.0" encoding="utf-8"?>
	<!DOCTYPE tsung SYSTEM "/usr/share/tsung/tsung-1.0.dtd" []>
	<tsung loglevel="warning">

	  <clients>
		<client host="load_test_machine" cpu="2" maxusers="30000000"/>
	  </clients>

	  <servers>
		<server host="web_server_to_test" port="80" type="tcp"/>
	  </servers>

	  <load>
		<arrivalphase phase="1" duration="1" unit="minute">
		  <users arrivalrate="5" unit="second"/>
		</arrivalphase>
	  </load>

	  <sessions>
		<session name="es_load" weight="1" type="ts_http">
		  <request>
		  <http url="/postdata/Information"
				  method="POST"
				  contents_from_file="test.json" />
		  </request>
		</session>
	  </sessions>
	</tsung>

	
Test Sample Json

	{
		"name":"ahmed",
		"age":30
	}

	

### Next Execute the command to start the service .

	tsung -f load.xml start 

This will start the service which will start hitting the server.
All logs will be available in `${HOME}/.tsung/log`

More information 

	https://engineering.helpshift.com/2014/tsung/