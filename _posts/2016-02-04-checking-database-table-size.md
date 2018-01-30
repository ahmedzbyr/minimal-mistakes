---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: Mysql Database Disk Usage.
category: ['Linux', 'Mysql', 'Centos']
tags: ['mysql', 'centos', 'linux']
---

We were running out of disk space on one of the databases server, we need to get information on what the current table/database usage was. Below are few commands for `mysql` server tables usages.

To get details of table.

	show table status from zabbix;

You can use this query to show the size of a table (although you need to substitute the variables first):

	SELECT 
	    table_name AS `Table`, 
	    round(((data_length + index_length) / 1024 / 1024), 2) `Size in MB` 
	FROM information_schema.TABLES 
	WHERE table_schema = "$DB_NAME"
	    AND table_name = "$TABLE_NAME";

Here is the output:

	mysql> SELECT 
	    ->     table_name AS `Table`, 
	    ->     round(((data_length + index_length) / 1024 / 1024), 2) `Size in MB` 
	    -> FROM information_schema.TABLES 
	    -> WHERE table_schema = "zabbix"
	    ->     AND table_name = "history";
	+---------+------------+
	| Table   | Size in MB |
	+---------+------------+
	| history |       0.03 |
	+---------+------------+
	1 row in set (0.00 sec)
	
	mysql> 



or this query to list the size of every table in every database, largest first:

	SELECT 
	     table_schema as `Database`, 
	     table_name AS `Table`, 
	     round(((data_length + index_length) / 1024 / 1024), 2) `Size in MB` 
	FROM information_schema.TABLES 
	ORDER BY (data_length + index_length) DESC;

Here is the output:

	mysql> SELECT 
	    ->      table_schema as `Database`, 
	    ->      table_name AS `Table`, 
	    ->      round(((data_length + index_length) / 1024 / 1024), 2) `Size in MB` 
	    -> FROM information_schema.TABLES 
	    -> ORDER BY (data_length + index_length) DESC; 
	+--------------------+---------------------------------------+------------+
	| Database           | Table                                 | Size in MB |
	+--------------------+---------------------------------------+------------+
	| zabbix             | items                                 |       3.41 |
	| zabbix             | auditlog                              |       3.30 |
	| zabbix             | functions                             |       3.17 |
	| zabbix             | triggers                              |       2.67 |
	| zabbix             | images                                |       1.53 |
	| zabbix             | items_applications                    |       0.64 |
	| zabbix             | events                                |       0.56 |
	| mysql              | help_topic                            |       0.46 |
	| zabbix             | housekeeper                           |       0.19 |
	| zabbix             | history_uint                          |       0.16 |
	| zabbix             | auditlog_details                      |       0.13 |
	| zabbix             | opconditions                          |       0.03 |
	| zabbix             | trigger_discovery                     |       0.03 |
	| zabbix             | sysmap_url                            |       0.03 |
	| zabbix             | proxy_autoreg_host                    |       0.03 |
	| zabbix             | dbversion                             |       0.02 |
	| zabbix             | zbxe_translation                      |       0.01 |
	...
	    < more verbose ... >
	...
	| mysql              | plugin                                |       0.00 |
	| mysql              | func                                  |       0.00 |
	| mysql              | time_zone_name                        |       0.00 |
	| mysql              | ndb_binlog_index                      |       0.00 |
	| mysql              | time_zone_leap_second                 |       0.00 |
	| information_schema | EVENTS                                |       0.00 |
	| information_schema | PROCESSLIST                           |       0.00 |
	| mysql              | time_zone                             |       0.00 |
	| information_schema | SESSION_VARIABLES                     |       0.00 |
	| information_schema | COLUMN_PRIVILEGES                     |       0.00 |
	| mysql              | slow_log                              |       0.00 |
	| information_schema | SESSION_STATUS                        |       0.00 |
	| information_schema | KEY_COLUMN_USAGE                      |       0.00 |
	| information_schema | SCHEMA_PRIVILEGES                     |       0.00 |
	| information_schema | COLLATION_CHARACTER_SET_APPLICABILITY |       0.00 |
	| information_schema | USER_PRIVILEGES                       |       0.00 |
	| information_schema | SCHEMATA                              |       0.00 |
	| information_schema | COLLATIONS                            |       0.00 |
	| information_schema | GLOBAL_VARIABLES                      |       0.00 |
	| mysql              | general_log                           |       0.00 |
	| information_schema | CHARACTER_SETS                        |       0.00 |
	| information_schema | TABLE_PRIVILEGES                      |       0.00 |
	| information_schema | GLOBAL_STATUS                         |       0.00 |
	| information_schema | REFERENTIAL_CONSTRAINTS               |       0.00 |
	| information_schema | TABLE_CONSTRAINTS                     |       0.00 |
	| information_schema | FILES                                 |       0.00 |
	| information_schema | PROFILING                             |       0.00 |
	| information_schema | TABLES                                |       0.00 |
	| information_schema | STATISTICS                            |       0.00 |
	| information_schema | ENGINES                               |       0.00 |
	+--------------------+---------------------------------------+------------+
	157 rows in set (0.16 sec)

Copied straight from [`stackoverflow`](http://stackoverflow.com/questions/9620198/how-to-get-the-sizes-of-the-tables-of-a-mysql-database) just in case I forget. 