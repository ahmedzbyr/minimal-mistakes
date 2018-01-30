---
title: Kafka Kerberos Enable and Testing.
category: ['Linux', 'Centos', 'Redhat', 'Cloudera', 'Kafka', 'Kerberos', 'Cluster']
tags: ['linux', 'centos', 'redhat', 'cloudera', 'kafka', 'kerberos', 'cluster']
---

Apache Kafka is a distributed streaming platform. Kafka 2.0 supports Kerberos authentication, Enabling Kerberos Authentication Using the Wizard on cloudera manager. [Courtesy - Apache Kafka](https://kafka.apache.org/intro)

Before we start a little about `kafka`.

**We think of a streaming platform as having three key capabilities:**

- It lets you publish and subscribe to streams of records. In this respect it is similar to a message queue or enterprise messaging system.
- It lets you store streams of records in a fault-tolerant way.
- It lets you process streams of records as they occur.

**What is Kafka good for?**

It gets used for two broad classes of application:

- Building real-time streaming data pipelines that reliably get data between systems or applications
- Building real-time streaming applications that transform or react to the streams of data

To understand how Kafka does these things, let's dive in and explore Kafka's capabilities from the bottom up.

**First a few concepts:**

- Kafka is run as a cluster on one or more servers.
- The Kafka cluster stores streams of records in categories called topics.
- Each record consists of a key, a value, and a timestamp.

**Kafka has four core APIs:**

- The Producer API allows an application to publish a stream of records to one or more Kafka topics.
- The Consumer API allows an application to subscribe to one or more topics and process the stream of records produced to them.
- The Streams API allows an application to act as a stream processor, consuming an input stream from one or more topics and producing an output stream to one or more output topics, effectively transforming the input streams to output streams.
- The Connector API allows building and running reusable producers or consumers that connect Kafka topics to existing applications or data systems. For example, a connector to a relational database might capture every change to a table.


## Kafka Kerberos Testing.

Let us get started with enabling kafka kerberos and test the setting. 

### Step 1: To enable Kerberos authentication for Kafka:

Follow the steps below. 

- Install the CDH Kafka v2.0.x parcel and Cloudera Manager 5.5.3 or higher. 
- Enable Kerberos using [Cloudera Manager](https://www.cloudera.com/documentation/enterprise/latest/topics/cm_sg_intro_kerb.html)
- From Cloudera Manager, navigate to `Kafka > Configurations`. 
- Set `SSL client authentication` to `none`. 
- Set `Inter Broker Protocol` to `SASL_PLAINTEXT`.
- Click Save Changes.
- Restart the `kafka` service.

### Step 2: Check logs if we see the `listeners = SASL_PLAINTEXT` set configuration set.

Follow below steps

- look for `listeners = SASL_PLAINTEXT` is present in the Kafka broker logs `/var/log/kafka/server.log`.

### Step 3: Create a `keytab` file for current login user. 

Creating a keytab.

	[ahmed@ahmed-server-kafka-001 ~]$ ktutil
	ktutil:  addent -password -p ahmed@AHMED.AHMEDINC.COM -k 1 -e RC4-HMAC
	Password for ahmed@AHMED.AHMEDINC.COM: ********
	ktutil:  wkt ahmed.keytab
	ktutil:  quit


### Step 4a: Create a `jaas.conf` in `$HOME` directory.

Add below configuration in the file. 

	[ahmed@ahmed-server-kafka-001 ~]$ cat jaas.conf
	KafkaClient {
	com.sun.security.auth.module.Krb5LoginModule required
	debug=true
	useTicketCache=true
	serviceName="kafka"
	renewTicker=true
	doNotPrompt=true
	client=true
	principal="ahmed@AHMED.AHMEDINC.COM"
	keyTab="/export/home/ahmed/ahmed.keytab"
	useKeyTab=true;
	};

### Step 4b: Create a `client.properties` file in `$HOME` directory.

Add below configuration in the file. 

	[ahmed@ahmed-server-kafka-001 ~]$ cat client.properties
	security.protocol=SASL_PLAINTEXT
	sasl.kerberos.service.name=kafka
	[ahmed@ahmed-server-kafka-001 ~]$

### Step 5: Adding `KAFKA_OPTS` to the environment.

Command to get the `KAFKA_OPTS` set.

	[ahmed@ahmed-server-kafka-001 ~]$ export KAFKA_OPTS="-Djava.security.auth.login.config=/export/home/ahmed/jaas.conf"         

### Step 6: Creating a `topic` on `kafka` cluster.	

Command to use. 

	kafka-topics --create --zookeeper ahmed-server-kafka-001:2181/kafka --replication-factor 3 --partitions 3 --topic test1

Output.

	[ahmed@ahmed-server-kafka-001 ~]$ kafka-topics --create --zookeeper ahmed-server-kafka-001:2181/kafka --replication-factor 3 --partitions 3 --topic test1
	...<verbose>...
	17/05/15 07:01:26 INFO zookeeper.ZooKeeper: Client environment:java.io.tmpdir=/tmp
	17/05/15 07:01:26 INFO zookeeper.ZooKeeper: Client environment:java.compiler=<NA>
	17/05/15 07:01:26 INFO zookeeper.ZooKeeper: Client environment:os.name=Linux
	17/05/15 07:01:26 INFO zookeeper.ZooKeeper: Client environment:os.arch=amd64
	17/05/15 07:01:26 INFO zookeeper.ZooKeeper: Client environment:os.version=2.6.32-642.6.2.el6.centos.plus.x86_64
	17/05/15 07:01:26 INFO zookeeper.ZooKeeper: Client environment:user.name=ahmed
	17/05/15 07:01:26 INFO zookeeper.ZooKeeper: Client environment:user.home=/export/home/ahmed
	17/05/15 07:01:26 INFO zookeeper.ZooKeeper: Client environment:user.dir=/export/home/ahmed
	17/05/15 07:01:26 INFO zookeeper.ZooKeeper: Initiating client connection, connectString=ahmed-server-kafka-001.tigris.equif  ax.com:2181/kafka sessionTimeout=30000 watcher=org.I0Itec.zkclient.ZkClient@27d9954b
	17/05/15 07:01:26 INFO zkclient.ZkClient: Waiting for keeper state SyncConnected
	17/05/15 07:01:26 INFO zookeeper.ClientCnxn: Opening socket connection to server ahmed-server-kafka-001.ahmedinc.com/  192.168.12.100:2181. Will not attempt to authenticate using SASL (unknown error)
	17/05/15 07:01:26 INFO zookeeper.ClientCnxn: Socket connection established to ahmed-server-kafka-001.ahmedinc.com/192  .168.12.100:2181, initiating session
	17/05/15 07:01:26 INFO zookeeper.ClientCnxn: Session establishment complete on server ahmed-server-kafka-001.ahmedinc.com/192.168.12.100:2181, sessionid = 0x655bfe5b19d311e5, negotiated timeout = 30000
	17/05/15 07:01:26 INFO zkclient.ZkClient: zookeeper state changed (SyncConnected)
	17/05/15 07:01:26 INFO admin.AdminUtils$: Topic creation {"version":1,"partitions":{"0":[274]}}
	Created topic "test1".
	17/05/15 07:01:26 INFO zkclient.ZkEventThread: Terminate ZkClient event thread.
	17/05/15 07:01:26 INFO zookeeper.ZooKeeper: Session: 0x655bfe5b19d311e5 closed
	17/05/15 07:01:26 INFO zookeeper.ClientCnxn: EventThread shut down


### Step 7: Checking for the created `topic`s.
	
Command to execute.
	
	kafka-topics --list --zookeeper ahmed-server-kafka-001:2181/kafka
	
Output.
	
	[ahmed@ahmed-server-kafka-001 ~]$ kafka-topics --list --zookeeper ahmed-server-kafka-001:2181/kafka
	17/05/15 08:51:13 INFO zkclient.ZkClient: JAAS File name: /export/home/ahmed/jaas.conf
	17/05/15 08:51:13 INFO zkclient.ZkEventThread: Starting ZkClient event thread.
	17/05/15 08:51:13 INFO zookeeper.ZooKeeper: Client environment:zookeeper.version=3.4.5-cdh5.4.0-SNAPSHOT--1, built on 04/01/2015 08:35 GMT
	17/05/15 08:51:13 INFO zookeeper.ZooKeeper: Client environment:host.name=ahmed-server-kafka-001.ahmedinc.com
	17/05/15 08:51:13 INFO zookeeper.ZooKeeper: Client environment:java.version=1.7.0_67
	17/05/15 08:51:13 INFO zookeeper.ZooKeeper: Client environment:java.vendor=Oracle Corporation
	17/05/15 08:51:13 INFO zookeeper.ZooKeeper: Client environment:java.home=/usr/java/jdk1.7.0_67-cloudera/jre
	17/05/15 08:51:13 INFO zookeeper.ZooKeeper: Client environment:java.class.path=:/opt/cloudera/parcels/KAFKA-2.0.2-1.2.0.2.p ...<verbose>... /bin/../libs/guava-18.0.jar
	17/05/15 08:51:13 INFO zookeeper.ZooKeeper: Client environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
	17/05/15 08:51:13 INFO zookeeper.ZooKeeper: Client environment:java.io.tmpdir=/tmp
	17/05/15 08:51:13 INFO zookeeper.ZooKeeper: Client environment:java.compiler=<NA>
	17/05/15 08:51:13 INFO zookeeper.ZooKeeper: Client environment:os.name=Linux
	17/05/15 08:51:13 INFO zookeeper.ZooKeeper: Client environment:os.arch=amd64
	17/05/15 08:51:13 INFO zookeeper.ZooKeeper: Client environment:os.version=2.6.32-642.6.2.el6.centos.plus.x86_64
	17/05/15 08:51:13 INFO zookeeper.ZooKeeper: Client environment:user.name=ahmed
	17/05/15 08:51:13 INFO zookeeper.ZooKeeper: Client environment:user.home=/export/home/ahmed
	17/05/15 08:51:13 INFO zookeeper.ZooKeeper: Client environment:user.dir=/export/home/ahmed
	17/05/15 08:51:13 INFO zookeeper.ZooKeeper: Initiating client connection, connectString=ahmed-server-kafka-001:2181/kafka sessionTimeout=30000 watcher=org.I0Itec.zkclient.ZkClient@1381ee41
	17/05/15 08:51:13 INFO zkclient.ZkClient: Waiting for keeper state SyncConnected
	17/05/15 08:51:13 WARN zookeeper.ClientCnxn: SASL configuration failed: javax.security.auth.login.LoginException: No JAAS configuration section named 'Client' was found in specified JAAS configuration file: '/export/home/ahmed/jaas.conf'. Will continue connection to Zookeeper server without SASL authentication, if Zookeeper server allows it.
	17/05/15 08:51:13 INFO zookeeper.ClientCnxn: Opening socket connection to server ahmed-server-kafka-001.ahmedinc.com/192.168.12.100:2181
	17/05/15 08:51:13 INFO zkclient.ZkClient: zookeeper state changed (AuthFailed)
	17/05/15 08:51:13 INFO zookeeper.ClientCnxn: Socket connection established to ahmed-server-kafka-001.ahmedinc.com/192.168.12.100:2181, initiating session
	17/05/15 08:51:13 INFO zookeeper.ClientCnxn: Session establishment complete on server ahmed-server-kafka-001.ahmedinc.com/192.168.12.100:2181, sessionid = 0x655bfe5b19d3127d, negotiated timeout = 30000
	17/05/15 08:51:13 INFO zkclient.ZkClient: zookeeper state changed (SyncConnected)
	__consumer_offsets
	test1
	testz
	17/05/15 08:51:13 INFO zkclient.ZkEventThread: Terminate ZkClient event thread.
	17/05/15 08:51:13 INFO zookeeper.ZooKeeper: Session: 0x655bfe5b19d3127d closed
	17/05/15 08:51:13 INFO zookeeper.ClientCnxn: EventThread shut down


### Step 8: Creating a console producer to generate messages. 

Command to execute.

	kafka-console-producer --broker-list ahmed-server-kafka-001.ahmedinc.com:9092 --topic test1 --producer.config client.properties

Output. 

	[ahmed@ahmed-server-kafka-001 ~]$ kafka-console-producer --broker-list ahmed-server-kafka-001.ahmedinc.com:9092 --topic test1 --producer.config client.properties
	17/05/15 08:34:13 INFO producer.ProducerConfig: ProducerConfig values:
			request.timeout.ms = 1500
			retry.backoff.ms = 100
			buffer.memory = 33554432
			ssl.truststore.password = null
			batch.size = 16384
			ssl.keymanager.algorithm = SunX509
			receive.buffer.bytes = 32768
			ssl.cipher.suites = null
			ssl.key.password = null
			sasl.kerberos.ticket.renew.jitter = 0.05
			ssl.provider = null
			sasl.kerberos.service.name = kafka
			max.in.flight.requests.per.connection = 5
			sasl.kerberos.ticket.renew.window.factor = 0.8
			bootstrap.servers = [ahmed-server-kafka-001.ahmedinc.com:9092]
			client.id = console-producer
			max.request.size = 1048576
			acks = 0
			linger.ms = 1000
			sasl.kerberos.kinit.cmd = /usr/bin/kinit
			ssl.enabled.protocols = [TLSv1.2, TLSv1.1, TLSv1]
			metadata.fetch.timeout.ms = 60000
			ssl.endpoint.identification.algorithm = null
			ssl.keystore.location = null
			value.serializer = class org.apache.kafka.common.serialization.ByteArraySerializer
			ssl.truststore.location = null
			ssl.keystore.password = null
			key.serializer = class org.apache.kafka.common.serialization.ByteArraySerializer
			block.on.buffer.full = false
			metrics.sample.window.ms = 30000
			metadata.max.age.ms = 300000
			security.protocol = SASL_PLAINTEXT
			ssl.protocol = TLS
			sasl.kerberos.min.time.before.relogin = 60000
			timeout.ms = 30000
			connections.max.idle.ms = 540000
			ssl.trustmanager.algorithm = PKIX
			metric.reporters = []
			compression.type = none
			ssl.truststore.type = JKS
			max.block.ms = 60000
			retries = 3
			send.buffer.bytes = 102400
			partitioner.class = class org.apache.kafka.clients.producer.internals.DefaultPartitioner
			reconnect.backoff.ms = 50
			metrics.num.samples = 2
			ssl.keystore.type = JKS

	Debug is  true storeKey false useTicketCache true useKeyTab true doNotPrompt true ticketCache is null isInitiator true KeyTab is /export/home/ahmed/ahmed.keytab refreshKrb5Config is false principal is ahmed@AHMED.AHMEDINC.COM tryFirstPass is false useFirstPass is false storePass is false clearPass is false
	Acquire TGT from Cache
	Principal is ahmed@AHMED.AHMEDINC.COM
	null credentials from Ticket Cache
	principal is ahmed@AHMED.AHMEDINC.COM
	Will use keytab
	Commit Succeeded

	17/05/15 08:34:14 INFO kerberos.Login: Successfully logged in.
	17/05/15 08:34:14 INFO kerberos.Login: TGT refresh thread started.
	17/05/15 08:34:14 INFO kerberos.Login: TGT valid starting at: Mon May 15 08:34:13 UTC 2017
	17/05/15 08:34:14 INFO kerberos.Login: TGT expires: Mon May 15 18:34:13 UTC 2017
	17/05/15 08:34:14 INFO kerberos.Login: TGT refresh sleeping until: Mon May 15 16:52:23 UTC 2017
	17/05/15 08:34:14 INFO utils.AppInfoParser: Kafka version : 0.9.0-kafka-2.0.2
	17/05/15 08:34:14 INFO utils.AppInfoParser: Kafka commitId : unknown
	Hello World
	How are you
	This is a test message from test1 Topic
	This is a test message 2 from test1 topic
	message 3
	^C17/05/15 08:37:05 INFO producer.KafkaProducer: Closing the Kafka producer with timeoutMillis = 9223372036854775807 ms.
	17/05/15 08:37:05 WARN kerberos.Login: TGT renewal thread has been interrupted and will exit.
	[ahmed@ahmed-server-kafka-001 ~]$


### Step 9: Create a duplicate session and login as a consumer. 	
	
Follow below steps.

1. Login to a different server (or create a duplicate session)
2. Set `KAFKA_OPTS` for the environment. 
3. Execute below command. 

Command.

	kafka-console-consumer --new-consumer --topic test1 --from-beginning --bootstrap-server ahmed-server-kafka-001.ahmedinc.com:9092 --consumer.config client.properties
	
Output. 
	
	[ahmed@ahmed-server-kafka-001 ~]$ kafka-console-consumer --new-consumer --topic test1 --from-beginning --bootstrap-server ahmed-server-kafka-001.ahmedinc.com:9092 --consumer.config client.properties
	17/05/15 08:36:21 INFO consumer.ConsumerConfig: ConsumerConfig values:
			request.timeout.ms = 40000
			check.crcs = true
			retry.backoff.ms = 100
			ssl.truststore.password = null
			ssl.keymanager.algorithm = SunX509
			receive.buffer.bytes = 65536
			ssl.cipher.suites = null
			ssl.key.password = null
			sasl.kerberos.ticket.renew.jitter = 0.05
			ssl.provider = null
			sasl.kerberos.service.name = kafka
			session.timeout.ms = 30000
			sasl.kerberos.ticket.renew.window.factor = 0.8
			bootstrap.servers = [ahmed-server-kafka-001.ahmedinc.com:9092]
			client.id =
			fetch.max.wait.ms = 500
			fetch.min.bytes = 1
			key.deserializer = class org.apache.kafka.common.serialization.ByteArrayDeserializer
			sasl.kerberos.kinit.cmd = /usr/bin/kinit
			auto.offset.reset = earliest
			value.deserializer = class org.apache.kafka.common.serialization.ByteArrayDeserializer
			ssl.enabled.protocols = [TLSv1.2, TLSv1.1, TLSv1]
			partition.assignment.strategy = [org.apache.kafka.clients.consumer.RangeAssignor]
			ssl.endpoint.identification.algorithm = null
			max.partition.fetch.bytes = 1048576
			ssl.keystore.location = null
			ssl.truststore.location = null
			ssl.keystore.password = null
			metrics.sample.window.ms = 30000
			metadata.max.age.ms = 300000
			security.protocol = SASL_PLAINTEXT
			auto.commit.interval.ms = 5000
			ssl.protocol = TLS
			sasl.kerberos.min.time.before.relogin = 60000
			connections.max.idle.ms = 540000
			ssl.trustmanager.algorithm = PKIX
			group.id = console-consumer-7657
			enable.auto.commit = true
			metric.reporters = []
			ssl.truststore.type = JKS
			send.buffer.bytes = 131072
			reconnect.backoff.ms = 50
			metrics.num.samples = 2
			ssl.keystore.type = JKS
			heartbeat.interval.ms = 3000

	Debug is  true storeKey false useTicketCache true useKeyTab true doNotPrompt true ticketCache is null isInitiator true KeyTab is /export/home/ahmed/ahmed.keytab refreshKrb5Config is false principal is ahmed@AHMED.AHMEDINC.COM tryFirstPass is false useFirstPass is false storePass is false clearPass is false
	Acquire TGT from Cache
	Principal is ahmed@AHMED.AHMEDINC.COM
	null credentials from Ticket Cache
	principal is ahmed@AHMED.AHMEDINC.COM
	Will use keytab
	Commit Succeeded

	17/05/15 08:36:21 INFO kerberos.Login: Successfully logged in.
	17/05/15 08:36:21 INFO kerberos.Login: TGT refresh thread started.
	17/05/15 08:36:21 INFO kerberos.Login: TGT valid starting at: Mon May 15 08:36:20 UTC 2017
	17/05/15 08:36:21 INFO kerberos.Login: TGT expires: Mon May 15 18:36:20 UTC 2017
	17/05/15 08:36:21 INFO kerberos.Login: TGT refresh sleeping until: Mon May 15 16:55:07 UTC 2017
	17/05/15 08:36:21 INFO utils.AppInfoParser: Kafka version : 0.9.0-kafka-2.0.2
	17/05/15 08:36:21 INFO utils.AppInfoParser: Kafka commitId : unknown
	17/05/15 08:36:22 INFO internals.AbstractCoordinator: Discovered coordinator ahmed-server-kafka-006.ahmedinc.com:9092 (id: 2147483367) for group console-consumer-7657.
	17/05/15 08:36:22 INFO internals.ConsumerCoordinator: Revoking previously assigned partitions [] for group console-consumer-7657
	17/05/15 08:36:22 INFO internals.AbstractCoordinator: (Re-)joining group console-consumer-7657
	17/05/15 08:36:22 INFO internals.AbstractCoordinator: Marking the coordinator ahmed-server-kafka-006.ahmedinc.com:9092 (id: 2147483367) dead for group console-consumer-7657
	17/05/15 08:36:22 INFO internals.AbstractCoordinator: Discovered coordinator ahmed-server-kafka-006.ahmedinc.com:9092 (id: 2147483367) for group console-consumer-7657.
	17/05/15 08:36:22 INFO internals.AbstractCoordinator: (Re-)joining group console-consumer-7657
	17/05/15 08:36:23 INFO internals.AbstractCoordinator: Successfully joined group console-consumer-7657 with generation 1
	17/05/15 08:36:23 INFO internals.ConsumerCoordinator: Setting newly assigned partitions [test1-0] for group console-consumer-7657
	Hello World
	How are you
	This is a test message from test1 Topic
	This is a test message 2 from test1 topic
	message 3
	^C17/05/15 08:37:10 WARN kerberos.Login: TGT renewal thread has been interrupted and will exit.
	Processed a total of 5 messages
	17/05/15 08:37:10 INFO zkclient.ZkClient: JAAS File name: /export/home/ahmed/jaas.conf
	17/05/15 08:37:10 INFO zkclient.ZkEventThread: Starting ZkClient event thread.
	17/05/15 08:37:10 INFO zookeeper.ZooKeeper: Client environment:zookeeper.version=3.4.5-cdh5.4.0-SNAPSHOT--1, built on 04/01/2015 08:35 GMT
	17/05/15 08:37:10 INFO zookeeper.ZooKeeper: Client environment:host.name=ahmed-server-kafka-001.ahmedinc.com
	17/05/15 08:37:10 INFO zookeeper.ZooKeeper: Client environment:java.version=1.7.0_67
	17/05/15 08:37:10 INFO zookeeper.ZooKeeper: Client environment:java.vendor=Oracle Corporation
	17/05/15 08:37:10 INFO zookeeper.ZooKeeper: Client environment:java.home=/usr/java/jdk1.7.0_67-cloudera/jre
	17/05/15 08:37:10 INFO zookeeper.ZooKeeper: Client environment:java.class.path=:/opt/cloudera/parcels/KAFKA-2.0...<verbose>... n/../libs/guava-18.0.jar
	17/05/15 08:37:10 INFO zookeeper.ZooKeeper: Client environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
	17/05/15 08:37:10 INFO zookeeper.ZooKeeper: Client environment:java.io.tmpdir=/tmp
	17/05/15 08:37:10 INFO zookeeper.ZooKeeper: Client environment:java.compiler=<NA>
	17/05/15 08:37:10 INFO zookeeper.ZooKeeper: Client environment:os.name=Linux
	17/05/15 08:37:10 INFO zookeeper.ZooKeeper: Client environment:os.arch=amd64
	17/05/15 08:37:10 INFO zookeeper.ZooKeeper: Client environment:os.version=2.6.32-642.6.2.el6.centos.plus.x86_64
	17/05/15 08:37:10 INFO zookeeper.ZooKeeper: Client environment:user.name=ahmed
	17/05/15 08:37:10 INFO zookeeper.ZooKeeper: Client environment:user.home=/export/home/ahmed
	17/05/15 08:37:10 INFO zookeeper.ZooKeeper: Client environment:user.dir=/export/home/ahmed
	17/05/15 08:37:10 INFO zookeeper.ZooKeeper: Initiating client connection, connectString=null sessionTimeout=30000 watcher=org.I0Itec.zkclient.ZkClient@26132588
	17/05/15 08:37:10 INFO zkclient.ZkEventThread: Terminate ZkClient event thread.

