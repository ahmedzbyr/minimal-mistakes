---
title: Basic Testing On Hadoop Environment [Cloudera]
category: ['Linux', 'Cloudera', 'Hadoop', 'Hue', 'Testing']
tags: ['linux', 'cloudera', 'hadoop', 'testing']
---

These are a set of testing which we can do on a Hadoop environment. These are basic testing to make sure the environment is setup correctly.

NOTE : On a `kerberized` cluster we need to use the keytab to execute these commands.

**Creating keytab.**

``` sh
$ ktutil
ktutil:  addent -password -p <userid>@ADDOMAIN.AHMEDINC.COM -k 1 -e RC4-HMAC
Password for <userid>@ADDOMAIN.AHMEDINC.COM: ********
ktutil:  wkt <userid>.keytab
ktutil:  quit
$ ls
<userid>.keytab
```

### HDFS Testing


**Running `pi`**

``` sh
hadoop jar /opt/cloudera/parcels/CDH/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar pi 100 100000
```

 **Running `TestDFSIO`**

``` sh
hadoop jar  /opt/cloudera/parcels/CDH/lib/hadoop-mapreduce/hadoop-mapreduce-client-jobclient*tests*.jar
```

Command output.

``` sh
$ hadoop jar  /opt/cloudera/parcels/CDH/lib/hadoop-mapreduce/hadoop-mapreduce-client-jobclient*tests*.jar
Unknown program '/opt/cloudera/parcels/CDH/lib/hadoop-mapreduce/hadoop-mapreduce-client-jobclient-tests.jar' chosen.
Valid program names are:
  DFSCIOTest: Distributed i/o benchmark of libhdfs.
  DistributedFSCheck: Distributed checkup of the file system consistency.
  JHLogAnalyzer: Job History Log analyzer.
  MRReliabilityTest: A program that tests the reliability of the MR framework by injecting faults/failures
  SliveTest: HDFS Stress Test and Live Data Verification.
  TestDFSIO: Distributed i/o benchmark.
  fail: a job that always fails
  filebench: Benchmark SequenceFile(Input|Output)Format (block,record compressed and uncompressed), Text(Input|Output)Format (compressed and uncompressed)
  largesorter: Large-Sort tester
  loadgen: Generic map/reduce load generator
  mapredtest: A map/reduce test check.
  minicluster: Single process HDFS and MR cluster.
  mrbench: A map/reduce benchmark that can create many small jobs
  nnbench: A benchmark that stresses the namenode.
  sleep: A job that sleeps at each map and reduce task.
  testbigmapoutput: A map/reduce program that works on a very big non-splittable file and does identity map/reduce
  testfilesystem: A test for FileSystem read/write.
  testmapredsort: A map/reduce program that validates the map-reduce framework's sort.
  testsequencefile: A test for flat files of binary key value pairs.
  testsequencefileinputformat: A test for sequence file input format.
  testtextinputformat: A test for text input format.
  threadedmapbench: A map/reduce benchmark that compares the performance of maps with multiple spills over maps with 1 spill

```


Example execution.

``` sh
hadoop jar  /opt/cloudera/parcels/CDH/lib/hadoop-mapreduce/hadoop-mapreduce-client-jobclient*tests*.jar TestDFSIO -write -nrFiles 10 -fileSize 1000
hadoop jar  /opt/cloudera/parcels/CDH/lib/hadoop-mapreduce/hadoop-mapreduce-client-jobclient*tests*.jar TestDFSIO -read -nrFiles 10 -fileSize 1000
```

**Running Terasort**

First create the data using `teragen`.

``` sh
hadoop jar /opt/cloudera/parcels/CDH/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar teragen 1000000 /user/zahmed/terasort-input
```

Then execute `terasort` (mapreduce job) on the generated `teragen` data set.

``` sh
hadoop jar /opt/cloudera/parcels/CDH/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar terasort /user/zahmed/terasort-input /user/zahmed/terasort-output
```

### YARN Testing

When the above jobs are running we can go to `Cloudera manager` -> `YARN` -> `Applications` to check the application running.


### Testing Hive from Hue

If using a kerberos environment do [this](https://zubayr.github.io/hive-access-permission/) before creating a table. 

Creating a Database.

``` sh
create database TEST;
``` 

Creating a Table.

``` sh
use TEST;
CREATE TABLE IF NOT EXISTS employee ( eid int, name String, salary String, destination String);
```

Insert into table.

``` sh
insert into table employee values (1,'zubair','13123123','eng')
select * from employee where eid=1;
```

This should return inserted value.


### Testing Impala from Hue

Invalidate metastore and check for `hive` database.

``` sh
invalidate metadata;
```

You should see the `test` database created earlier. Execute `select` query to verify.

``` sh
select * from employee where eid=1;
```

### Testing Spark

Running a Pi Job. Logon to one of the Gateway nodes.

``` sh
spark-submit --class org.apache.spark.examples.SparkPi --deploy-mode cluster --master yarn /opt/cloudera/parcels/CDH-5.8.3-1.cdh5.8.3.p0.2/lib/spark/lib/spark-examples.jar 10
```

[https://www.cloudera.com/documentation/enterprise/5-3-x/topics/cdh_ig_running_spark_apps.html](https://www.cloudera.com/documentation/enterprise/5-3-x/topics/cdh_ig_running_spark_apps.html)

### Testing and Grant Permission on Hbase


First pick the hbase keytab above and execute below command.
NOTE: If you are using a `kerberos` environment and want to give access to other users, you need to use the `hbase` keytab.

``` sh
$ hbase shell
17/02/20 08:44:29 INFO Configuration.deprecation: hadoop.native.lib is deprecated. Instead, use io.native.lib.available
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Vaddomainion 1.2.0-cdh5.8.3, rUnknown, Wed Oct 12 20:32:08 PDT 2016
```

Creating `emp` table.

``` sh
hbase(main):001:0> create 'emp', 'paddomainonal data', 'professional data'
0 row(s) in 2.5390 seconds

=> Hbase::Table - emp
hbase(main):002:0> list
TABLE
emp
1 row(s) in 0.0120 seconds

=> ["emp"]
hbase(main):003:0> user_permission emp
NameError: undefined local variable or method `emp' for #<Object:0x77752a85>
```

Checking user permission on the table, currently we have `hbase` user as the owner

``` sh
hbase(main):004:0> user_permission "emp"
User                                        Namespace,Table,Family,Qualifier:Permission
 hbase                                      default,emp,,: [Permission: actions=READ,WRITE,EXEC,CREATE,ADMIN]
1 row(s) in 0.3380 seconds
```

Adding permission to new user.

``` sh
hbase(main):005:0> grant "zahmed", "RWC", "emp"
0 row(s) in 0.2320 seconds
```

Checking Permission.

``` sh
hbase(main):006:0> user_permission "emp"
User                                        Namespace,Table,Family,Qualifier:Permission
 zahmed                                      default,emp,,: [Permission: actions=READ,WRITE,CREATE]
 hbase                                      default,emp,,: [Permission: actions=READ,WRITE,EXEC,CREATE,ADMIN]
2 row(s) in 0.0510 seconds

hbase(main):007:0>
```

Now logon to `hue` to check the new `hbase` table appear there.

### Testing SQOOP

Create a mysql database and add table with data.

Creating database.

``` sh
mysql> create database employee;
Query OK, 1 row affected (0.01 sec)
``` 

Creating Table.

``` sh
mysql> CREATE TABLE IF NOT EXISTS employees ( eid varchar(20), name varchar(25), salary varchar(20), destination varchar(15));
Query OK, 0 rows affected (0.00 sec)

mysql> show tables;
+--------------------+
| Tables_in_employee |
+--------------------+
| employees          |
+--------------------+
1 row in set (0.00 sec)


mysql> describe employees;
+-------------+-------------+------+-----+---------+-------+
| Field       | Type        | Null | Key | Default | Extra |
+-------------+-------------+------+-----+---------+-------+
| eid         | varchar(20) | YES  |     | NULL    |       |
| name        | varchar(25) | YES  |     | NULL    |       |
| salary      | varchar(20) | YES  |     | NULL    |       |
| destination | varchar(15) | YES  |     | NULL    |       |
+-------------+-------------+------+-----+---------+-------+
4 rows in set (0.00 sec)

```

Inserting data into the table.

``` sh
mysql> insert into employees values ("123EFD", "ZUBAIR AHMED", "1000", "ENGINEER");
Query OK, 1 row affected (0.00 sec)
```

Checking table.

``` sh
mysql> select * from employees;
+--------+--------------+--------+-------------+
| eid    | name         | salary | destination |
+--------+--------------+--------+-------------+
| 123EFD | ZUBAIR AHMED | 1000   | ENGINEER    |
+--------+--------------+--------+-------------+
1 row in set (0.01 sec)

mysql> insert into employees values ("123EFD123", "Z AHMED", "11000", "ENGINEER");
Query OK, 1 row affected (0.00 sec)

mysql> insert into employees values ("123123EFD123", "Z AHMD", "11000", "ENGINEER");
Query OK, 1 row affected (0.00 sec)

mysql> select * from employees;
+--------------+--------------+--------+-------------+
| eid          | name         | salary | destination |
+--------------+--------------+--------+-------------+
| 123EFD       | ZUBAIR AHMED | 1000   | ENGINEER    |
| 123EFD123    | Z AHMED      | 11000  | ENGINEER    |
| 123123EFD123 | Z AHMD       | 11000  | ENGINEER    |
+--------------+--------------+--------+-------------+
3 rows in set (0.00 sec)
```

Grant permission to a user which can access the database.

``` sh
mysql> grant all privileges on employee.* to emp@'%' identified by 'emp@123';
Query OK, 0 rows affected (0.00 sec)
```

Once we have the database created, execute command below.

``` sh
sqoop import --connect jdbc:mysql://atlbdl1drlha001.gpsbd.lab1.ahmedinc.com/employee --username emp --password emp@123 --query 'SELECT * from employees where $CONDITIONS' --split-by eid --target-dir /user/zahmed/sqoop_test
```

Command output.

``` sh
$ sqoop import --connect jdbc:mysql://atlbdl1drlha001.gpsbd.lab1.ahmedinc.com/employee --username emp --password emp@123 --query 'SELECT * from employees where $CONDITIONS' --split-by eid --target-dir /user/zahmed/sqoop_test
Warning: /opt/cloudera/parcels/CDH-5.8.3-1.cdh5.8.3.p0.2/bin/../lib/sqoop/../accumulo does not exist! Accumulo imports will fail.
Please set $ACCUMULO_HOME to the root of your Accumulo installation.
17/02/21 08:54:15 INFO sqoop.Sqoop: Running Sqoop vaddomainion: 1.4.6-cdh5.8.3
17/02/21 08:54:15 WARN tool.BaseSqoopTool: Setting your password on the command-line is insecure. Consider using -P instead.
17/02/21 08:54:16 INFO manager.MySQLManager: Preparing to use a MySQL streaming resultset.
17/02/21 08:54:16 INFO tool.CodeGenTool: Beginning code generation
17/02/21 08:54:16 INFO manager.SqlManager: Executing SQL statement: SELECT * from employees where  (1 = 0)
17/02/21 08:54:16 INFO manager.SqlManager: Executing SQL statement: SELECT * from employees where  (1 = 0)
17/02/21 08:54:16 INFO manager.SqlManager: Executing SQL statement: SELECT * from employees where  (1 = 0)
17/02/21 08:54:16 INFO orm.CompilationManager: HADOOP_MAPRED_HOME is /opt/cloudera/parcels/CDH/lib/hadoop-mapreduce
Note: /tmp/sqoop-cmadmin/compile/32f74db698040b57c22af35843d5af89/QueryResult.java uses or overrides a deprecated API.
Note: Recompile with -Xlint:deprecation for details.
17/02/21 08:54:17 INFO orm.CompilationManager: Writing jar file: /tmp/sqoop-cmadmin/compile/32f74db698040b57c22af35843d5af89/QueryResult.jar
17/02/21 08:54:17 INFO mapreduce.ImportJobBase: Beginning query import.
17/02/21 08:54:17 INFO Configuration.deprecation: mapred.jar is deprecated. Instead, use mapreduce.job.jar
17/02/21 08:54:18 INFO Configuration.deprecation: mapred.map.tasks is deprecated. Instead, use mapreduce.job.maps
17/02/21 08:54:18 INFO hdfs.DFSClient: Created token for zahmed: HDFS_DELEGATION_TOKEN owner=zahmed@ADDOMAIN.AHMEDINC.COM, renewer=yarn, realUser=, issueDate=1487667258619, maxDate=1488272058619, sequenceNumber=19, masterKeyId=10 on ha-hdfs:hdfsHA
17/02/21 08:54:18 INFO security.TokenCache: Got dt for hdfs://hdfsHA; Kind: HDFS_DELEGATION_TOKEN, Service: ha-hdfs:hdfsHA, Ident: (token for zahmed: HDFS_DELEGATION_TOKEN owner=zahmed@ADDOMAIN.AHMEDINC.COM, renewer=yarn, realUser=, issueDate=1487667258619, maxDate=1488272058619, sequenceNumber=19, masterKeyId=10)
17/02/21 08:54:20 INFO db.DBInputFormat: Using read commited transaction isolation
17/02/21 08:54:20 INFO db.DataDrivenDBInputFormat: BoundingValsQuery: SELECT MIN(eid), MAX(eid) FROM (SELECT * from employees where  (1 = 1) ) AS t1
17/02/21 08:54:20 WARN db.TextSplitter: Generating splits for a textual index column.
17/02/21 08:54:20 WARN db.TextSplitter: If your database sorts in a case-insensitive order, this may result in a partial import or duplicate records.
17/02/21 08:54:20 WARN db.TextSplitter: You are strongly encouraged to choose an integral split column.
17/02/21 08:54:20 INFO mapreduce.JobSubmitter: number of splits:5
17/02/21 08:54:20 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1487410266772_0001
17/02/21 08:54:20 INFO mapreduce.JobSubmitter: Kind: HDFS_DELEGATION_TOKEN, Service: ha-hdfs:hdfsHA, Ident: (token for zahmed: HDFS_DELEGATION_TOKEN owner=zahmed@ADDOMAIN.AHMEDINC.COM, renewer=yarn, realUser=, issueDate=1487667258619, maxDate=1488272058619, sequenceNumber=19, masterKeyId=10)
17/02/21 08:54:22 INFO impl.YarnClientImpl: Application submission is not finished, submitted application application_1487410266772_0001 is still in NEW
17/02/21 08:54:23 INFO impl.YarnClientImpl: Submitted application application_1487410266772_0001
17/02/21 08:54:23 INFO mapreduce.Job: The url to track the job: http://atlbdl1drlha001.gpsbd.lab1.ahmedinc.com:8088/proxy/application_1487410266772_0001/
17/02/21 08:54:23 INFO mapreduce.Job: Running job: job_1487410266772_0001
17/02/21 08:54:34 INFO mapreduce.Job: Job job_1487410266772_0001 running in uber mode : false
17/02/21 08:54:34 INFO mapreduce.Job:  map 0% reduce 0%
17/02/21 08:54:40 INFO mapreduce.Job:  map 20% reduce 0%
17/02/21 08:54:43 INFO mapreduce.Job:  map 60% reduce 0%
17/02/21 08:54:46 INFO mapreduce.Job:  map 100% reduce 0%
17/02/21 08:54:46 INFO mapreduce.Job: Job job_1487410266772_0001 completed successfully
17/02/21 08:54:46 INFO mapreduce.Job: Countaddomain: 30
        File System Countaddomain
                FILE: Number of bytes read=0
                FILE: Number of bytes written=768050
                FILE: Number of read operations=0
                FILE: Number of large read operations=0
                FILE: Number of write operations=0
                HDFS: Number of bytes read=636
                HDFS: Number of bytes written=102
                HDFS: Number of read operations=20
                HDFS: Number of large read operations=0
                HDFS: Number of write operations=10
        Job Countaddomain
                Launched map tasks=5
                Other local map tasks=5
                Total time spent by all maps in occupied slots (ms)=37208
                Total time spent by all reduces in occupied slots (ms)=0
                Total time spent by all map tasks (ms)=37208
                Total vcore-seconds taken by all map tasks=37208
                Total megabyte-seconds taken by all map tasks=38100992
        Map-Reduce Framework
                Map input records=3
                Map output records=3
                Input split bytes=636
                Spilled Records=0
                Failed Shuffles=0
                Merged Map outputs=0
                GC time elapsed (ms)=94
                CPU time spent (ms)=3680
                Physical memory (bytes) snapshot=1625182208
                Virtual memory (bytes) snapshot=8428191744
                Total committed heap usage (bytes)=4120903680
        File Input Format Countaddomain
                Bytes Read=0
        File Output Format Countaddomain
                Bytes Written=102
17/02/21 08:54:46 INFO mapreduce.ImportJobBase: Transferred 102 bytes in 27.8888 seconds (3.6574 bytes/sec)
17/02/21 08:54:46 INFO mapreduce.ImportJobBase: Retrieved 3 records.
```

 Checking for data in HDFS.

``` sh
$ hdfs dfs -ls /user/zahmed/
Found 2 items
drwx------   - zahmed supergroup          0 2017-02-21 08:54 /user/zahmed/.staging
drwxr-xr-x   - zahmed supergroup          0 2017-02-21 08:54 /user/zahmed/sqoop_test
```

Here is the data which was picked up by the (`SQOOP`) MR job.

``` sh
$ hdfs dfs -ls /user/zahmed/sqoop_test
Found 6 items
-rw-r--r--   3 zahmed supergroup          0 2017-02-21 08:54 /user/zahmed/sqoop_test/_SUCCESS
-rw-r--r--   3 zahmed supergroup          0 2017-02-21 08:54 /user/zahmed/sqoop_test/part-m-00000
-rw-r--r--   3 zahmed supergroup         35 2017-02-21 08:54 /user/zahmed/sqoop_test/part-m-00001
-rw-r--r--   3 zahmed supergroup          0 2017-02-21 08:54 /user/zahmed/sqoop_test/part-m-00002
-rw-r--r--   3 zahmed supergroup          0 2017-02-21 08:54 /user/zahmed/sqoop_test/part-m-00003
-rw-r--r--   3 zahmed supergroup         67 2017-02-21 08:54 /user/zahmed/sqoop_test/part-m-00004
$ hdfs dfs -cat /user/zahmed/sqoop_test/part-m-00000
$ hdfs dfs -cat /user/zahmed/sqoop_test/part-m-00001
123123EFD123,Z AHMD,11000,ENGINEER
$ hdfs dfs -cat /user/zahmed/sqoop_test/part-m-00003
$ hdfs dfs -cat /user/zahmed/sqoop_test/part-m-00002
$ hdfs dfs -cat /user/zahmed/sqoop_test/part-m-00004
123EFD,ZUBAIR AHMED,1000,ENGINEER
123EFD123,Z AHMED,11000,ENGINEER
```

[Note: Few of the jobs did not recieve any data as there were only 3 row in the table.]

### Key Trustee Testing

NOTE: To enable key trustee the server should be `kerberos` enabled.

#### Create a key and directory.

``` sh
kinit <KEY_ADMIN_USER>
hadoop key create mykey1
hadoop fs -mkdir /tmp/zone1
```

#### Create a zone and link to the key.

``` sh
kinit hdfs
hdfs crypto -createZone -keyName mykey1 -path /tmp/zone1
```

#### Create a file, put it in your zone and ensure the file can be decrypted.

``` sh
kinit <KEY_ADMIN_USER>
echo "Hello World" > /tmp/helloWorld.txt
hadoop fs -put /tmp/helloWorld.txt /tmp/zone1
hadoop fs -cat /tmp/zone1/helloWorld.txt
rm /tmp/helloWorld.txt
```

#### Ensure the file is stored as encrypted.

``` sh
kinit hdfs
hadoop fs -cat /.reserved/raw/tmp/zone1/helloWorld.txt
hadoop fs -rm -R /tmp/zone1
```

#### Command Output

Getting user credentials.

``` sh
$ kinit zahmed@ADDOMAIN.AHMEDINC.COM
Password for zahmed@ADDOMAIN.AHMEDINC.COM:
$ hdfs dfs -ls /
Found 3 items
drwx------   - hbase hbase               0 2017-02-23 14:43 /hbase
drwxrwxrwx   - hdfs  supergroup          0 2017-02-21 13:37 /tmp
drwxr-xr-x   - hdfs  supergroup          0 2017-02-17 17:47 /user
$ hdfs dfs -ls /user
Found 10 items
drwxr-xr-x   - hdfs   supergroup          0 2017-02-17 09:18 /user/hdfs
drwxrwxrwx   - mapred hadoop              0 2017-02-16 15:13 /user/history
drwxr-xr-x   - hdfs   supergroup          0 2017-02-17 19:15 /user/hive
drwxrwxr-x   - hue    hue                 0 2017-02-16 15:16 /user/hue
drwxrwxr-x   - impala impala              0 2017-02-16 15:16 /user/impala
drwxrwxr-x   - oozie  oozie               0 2017-02-16 15:17 /user/oozie
drwxr-x--x   - spark  spark               0 2017-02-16 15:14 /user/spark
drwxrwxr-x   - sqoop2 sqoop               0 2017-02-16 15:18 /user/sqoop2
drwxr-xr-x   - yxc27  supergroup          0 2017-02-17 18:09 /user/yxc27
drwxr-xr-x   - zahmed  supergroup          0 2017-02-20 08:20 /user/zahmed
```


 Creating a `key`

``` sh
$ hadoop key create mykey1
mykey1 has been successfully created with options Options{cipher='AES/CTR/NoPadding', bitLength=128, description='null', attributes=null}.
org.apache.hadoop.crypto.key.kms.LoadBalancingKMSClientProvider@62e10dd0 has been updated.
```

Creating a `zone`

``` sh
$ hadoop fs -mkdir /tmp/zone1
```

Login in as `hdfs`

``` sh
$ cd /var/run/cloudera-scm-agent/process/
$ sudo su
# ls -lt | grep hdfs
drwxr-x--x. 3 hdfs      hdfs      500 Feb 23 14:50 1071-namenodes-failover
drwxr-x--x. 3 hdfs      hdfs      500 Feb 23 14:48 1070-hdfs-NAMENODE-safemode-wait
drwxr-x--x. 3 hdfs      hdfs      380 Feb 23 14:47 1069-hdfs-FAILOVERCONTROLLER
drwxr-x--x. 3 hdfs      hdfs      400 Feb 23 14:47 598-hdfs-FAILOVERCONTROLLER
drwxr-x--x. 3 hdfs      hdfs      500 Feb 23 14:47 1068-hdfs-NAMENODE-nnRpcWait
drwxr-x--x. 3 hdfs      hdfs      500 Feb 23 14:47 1067-hdfs-NAMENODE
drwxr-x--x. 3 hdfs      hdfs      520 Feb 23 14:47 1063-hdfs-NAMENODE-rollEdits
drwxr-x--x. 3 hdfs      hdfs      500 Feb 23 14:47 1065-hdfs-NAMENODE-jnSyncWait
# cd 1071-namenodes-failover
# hostname
server.tigris.ahmedinc.com
# kinit -kt hdfs.keytab hdfs/server.tigris.ahmedinc.com@DEVDOMAIN.AHMEDINC.COM
```

Creating Zone.

``` sh
# hdfs crypto -createZone -keyName mykey1 -path /tmp/zone1
Added encryption zone /tmp/zone1
# exit
exit
```

Login in as `admin` user.

``` sh
$ klist
Ticket cache: FILE:/tmp/krb5cc_9002
Default principal: zahmed@ADDOMAIN.AHMEDINC.COM

Valid starting     Expires            Service principal
02/23/17 15:54:57  02/24/17 01:55:01  krbtgt/ADDOMAIN.AHMEDINC.COM@ADDOMAIN.AHMEDINC.COM
        renew until 03/02/17 15:54:57
$ echo "Hello World" > /tmp/helloWorld.txt
$ hadoop fs -put /tmp/helloWorld.txt /tmp/zone1
$ hadoop fs -cat /tmp/zone1/helloWorld.txt
Hello World
$ rm /tmp/helloWorld.txt
$ sudo su
# klist
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: hdfs/server.tigris.ahmedinc.com@DEVDOMAIN.AHMEDINC.COM

Valid starting     Expires            Service principal
02/23/17 15:57:15  02/24/17 01:57:14  krbtgt/DEVDOMAIN.AHMEDINC.COM@DEVDOMAIN.AHMEDINC.COM
        renew until 03/02/17 15:57:15
# hadoop fs -cat /.reserved/raw/tmp/zone1/helloWorld.txt
▒▒▒i▒
# hadoop fs -rm -R /tmp/zone1
17/02/23 15:58:59 INFO fs.TrashPolicyDefault: Moved: 'hdfs://hdfsHA/tmp/zone1' to trash at: hdfs://hdfsHA/user/hdfs/.Trash/Current/tmp/zone1
#
```