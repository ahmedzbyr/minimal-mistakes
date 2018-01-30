---
layout: post
title: Enable Authorization on HBase. 
category: hadoop
pdf: true
tags: hadoop hbase hadoop-config hbase-config security authentication
---

Add this below tag to all the Master and Region Server.

	<!-- Addding Authorization -->
	  <property>
		<name>hbase.security.authorization</name>
		<value>true</value>
	  </property>
	  <property>
		<name>hbase.coprocessor.master.classes</name>
		<value>org.apache.hadoop.hbase.security.access.AccessController</value>
	  </property>
	  <property>
		<name>hbase.coprocessor.region.classes</name>
		<value>org.apache.hadoop.hbase.security.token.TokenProvider,
                org.apache.hadoop.hbase.security.access.AccessController</value>
	  </property>
	  <property>
		<name>hbase.rpc.engine</name>
		<value>org.apache.hadoop.hbase.ipc.SecureRpcEngine</value>
	  </property>
	<!--End Authorization -->

Use the below command to grant access to hbase tables.

	[ahmed@master home]# sudo -u hbase hbase shell
	hbase(main):002:0> grant

	ERROR: wrong number of arguments (0 for 2)

	Here is some help for this command:
	Grant users specific rights.
	Syntax : grant <user> <permissions> [<table> [<column family> [<column qualifier>]]

	permissions is either zero or more letters from the set "RWXCA".
	READ('R'), WRITE('W'), EXEC('X'), CREATE('C'), ADMIN('A')

For example:

    hbase> grant 'bobsmith', 'RWXCA'
    hbase> grant 'bobsmith', 'RW', 't1', 'f1', 'col1'


	hbase(main):003:0> grant  'ahmed', 'RWCA'
	0 row(s) in 3.5870 seconds
 
	hbase(main):004:0> quit

### Helpful URLs.

More Information can be found in this links. 

* [CDH4 Security Guide](http://www.cloudera.com/content/cloudera/en/documentation/cdh4/v4-3-0/CDH4-Security-Guide/cdh4sg_topic_8_3.html)
* [Apache Mail Archives](http://mail-archives.apache.org/mod_mbox/hbase-user/201406.mbox/%3CCAOEq2C6KXB1q=tPNezPvTcrqFKrU29C4HkmjzCkfxvxYNQNcXw@mail.gmail.com%3E)
