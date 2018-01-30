---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: Zabbix Installation 2.4 - CentOS 6.5
category: ['Linux', 'Zabbix', 'Nagios', 'Monitoring']
tags: ['zabbix', 'centos', 'nagios', 'monitoring']
---

Zabbix is the ultimate enterprise-level software designed for monitoring availability and performance of IT infrastructure components. Zabbix is open source and comes at no cost.

Adding Repos and Install Server Configuration.

	[ahmed@server ~]# rpm -ivh \
	http://repo.zabbix.com/zabbix/2.4/rhel/6/x86_64/zabbix-release-2.4-1.el6.noarch.rpm
	[ahmed@server ~]# yum install mysql-server mysql
	[ahmed@server ~]# yum install zabbix-server-mysql zabbix-web-mysql
	[ahmed@server ~]# yum install zabbix-agent

Create Database.

	[ahmed@server ~]# mysql -uroot
	mysql> create database zabbix character set utf8 collate utf8_bin;
	mysql> grant all privileges on zabbix.* to zabbix@localhost identified by 'zabbix';
	mysql> exit

Creating Schema.

	[ahmed@server ~]# cd /usr/share/doc/zabbix-server-mysql-2.4.0/create
	[ahmed@server ~]# mysql -uroot zabbix < schema.sql
	[ahmed@server ~]# mysql -uroot zabbix < images.sql
	[ahmed@server ~]# mysql -uroot zabbix < data.sql

Setting Configuration.

	[ahmed@server ~]# vi /etc/zabbix/zabbix_server.conf
	DBHost=localhost
	DBName=zabbix
	DBUser=zabbix
	DBPassword=zabbix

Start Server.

	[ahmed@server ~]# service zabbix-server start

Apache Configuration. Change timezone.

	[ahmed@server ~]# sudo vim /etc/httpd/conf.d/zabbix.conf
	php_value max_execution_time 300
	php_value memory_limit 128M
	php_value post_max_size 16M
	php_value upload_max_filesize 2M
	php_value max_input_time 300
	php_value date.timezone Asia/Kolkata

Start httpd.

	[ahmed@server ~]# service httpd restart