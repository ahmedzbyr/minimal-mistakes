---
title: Setting up Local HBase on Top of HDFS.
category: ['Hadoop']
tags: ['hadoop', 'hdfs', 'hbase', 'hadoop-config', 'hbase-config']
---

First lets setup the Hbase Configuration Files. For pseudo-distributed replace <server_ip_address> with 'localhost'

    --------------
    hbase-site.xml
    --------------

	<configuration>
	  <property>
		<name>hbase.rootdir</name>
		<value>hdfs://<server_ip_address>:9000/hbase</value>
	  </property>
	 <property>
	 <name>hbase.zookeeper.property.clientPort</name>
	 <value>2181</value>
	 </property>
	  <property>
		<name>hbase.cluster.distributed</name>
		<value>true</value>
	  </property>
	  <property>
		  <name>hbase.zookeeper.quorum</name>
		  <value><server_ip_address></value>
	   </property>
	</configuration>


If  you are running zookeeper seperatly then make the line below as 'false'.
    
    --------------
    hbase-env.sh
    --------------

	export HBASE_MANAGES_ZK=false


Next creating a zoo.cfg for the zookeeper.

    ------------
    zoo.cfg
    ------------

	#  The number of milliseconds of each tick
	tickTime=2000
	#  The number of ticks that the initial
	#  synchronization phase can take
	initLimit=20
	#  The number of ticks that can pass between
	#  sending a request and getting an acknowledgement
	syncLimit=10
	#  the directory where the snapshot is stored.
	dataDir=/opt/mapr/zkdata
	#  the port at which the clients will connect
	clientPort=2181
	#  max number of client connections
	maxClientCnxns=100
	maxSessionTimeout=300000

Now lets create the HDFS configuration files.

    -------------
    core-site.xml
    -------------

	<configuration>
		<property>
			<name>fs.default.name</name>
			<value>hdfs://<server_ip_address>:9000</value>
		</property>
	</configuration>


    -------------
    hdfs.site.xml
    -------------

	<configuration>
		<property>
			<name>dfs.replication</name>
			<value>1</value>
		</property>

		<!--The path in this needs to be created first-->
		<property>
			<name>dfs.namenode.name.dir</name>
			<value>file:///root/hadoop-2.5.1/yarn_data/hdfs/namenode</value>
		</property>

		<!--The path in this needs to be created first-->
		<property>
			<name>dfs.datanode.data.dir</name>
			<value>file:///root/hadoop-2.5.1/yarn_data/hdfs/datanode</value>
		</property>
	</configuration>

    -------------
    mapred-site.xml
    -------------
	
	<configuration>
	<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>
	</configuration>


Add '<server_ip_address>' in slaves files.


    --------------
    yarn-site.xml
    --------------

	<configuration>
	<!-- Site specific YARN configuration properties -->
	<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
	</property>
	<property>
		<name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
		<value>org.apache.hadoop.mapred.ShuffleHandler</value>
	</property>
	</configuration>


Now That we are ready lets start the services.

This will start DFS (NameNode, SecondaryNameNode, DataNode)

	ahmed@master:~ ]# ./start-dfs.sh

This will start Yarn service (Resource Manager and Node Manager)
	
	ahmed@master:~ ]# ./start-yarn.sh

Here is the command output.

	ahmed@master:~ ]# jps
	43655 Jps
	12018 Bootstrap
	31585 NameNode
	32114 SecondaryNameNode
	31798 DataNode
	32494 NodeManager
	32277 ResourceManager

Lets Do a HDFS Test.

	./bin/hadoop jar \
            /root/hadoop-2.5.1/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.5.1-tests.jar \
                TestDFSIO -write -nrFiles 10 -fileSize 100


Next Lets start HBase services.

First Start the zookeeper.

	ahmed@master:~]# ./hbase-daemon.sh --config ../conf/zoo.cfg start zookeeper

Next start RegionServer

	ahmed@master:~]# ./hbase-daemon.sh start regionserver

Then we start the master

	ahmed@master:~]# ./hbase-daemon.sh start master

	ahmed@master:~] #  jps
	43655 Jps
	12018 Bootstrap
	40171 HQuorumPeer
	40425 HRegionServer
	31585 NameNode
	32114 SecondaryNameNode
	41509 HMaster
	31798 DataNode
	32494 NodeManager
	32277 ResourceManager

	root@SIDCLB:~/hadoop/etc/hadoop#  hbase shell
	2014-12-30 20:04:15,116 INFO  [main] Configuration.deprecation: hadoop.native.lib is deprecated.\
                        Instead, use io.native.lib.available
	HBase Shell; enter 'help<RETURN>' for list of supported commands.
	Type "exit<RETURN>" to leave the HBase Shell
	Version 0.98.1-hadoop2, r1583035, Sat Mar 29 17:19:25 PDT 2014

	hbase(main):001:0> list
	TABLE
	test
	1 row(s) in 2.3090 seconds

	=> ["test"]
	hbase(main):002:0> scan 'test'
	ROW                                  COLUMN+CELL
	 row                                 column=test_fam:, timestamp=1419947982152, value=NewValue
	1 row(s) in 0.4890 seconds

	hbase(main):003:0> put 'test', 'row2', 'test_fam', 'SecondValue'
	0 row(s) in 0.1110 seconds

	hbase(main):004:0> scan 'test'
	ROW                                  COLUMN+CELL
	 row                                 column=test_fam:, timestamp=1419947982152, value=NewValue
	 row2                                column=test_fam:, timestamp=1419950094363, value=SecondValue
	2 row(s) in 0.0260 seconds

	hbase(main):005:0>


[MapR](http://answers.mapr.com/questions/2514/hbase-code-is-not-running-java-net-connectexception.html)

[Hbase Standalone](http://hbase.apache.org/book/standalone_dist.html)

[Hbase Quick Start](http://hbase.apache.org/book/quickstart.html)

[Connection Refused](http://wiki.apache.org/hadoop/ConnectionRefused)

[Stack Overflow](http://stackoverflow.com/questions/22663484/get-error-cant-get-master-address-from-zookeeper-znode-data-null-when-us)

[Apache Zookeeper](http://hbase.apache.org/book/zookeeper.html# d248e13764)

[Apache Hadoop Yarn](http://hadoop.apache.org/docs/r2.3.0/hadoop-yarn/hadoop-yarn-site/YARN.html)

[Stack Overflow](http://stackoverflow.com/questions/24207493/hbase-on-hortonworks-hdp-sandbox-cant-get-master-address-from-zookeeper)



