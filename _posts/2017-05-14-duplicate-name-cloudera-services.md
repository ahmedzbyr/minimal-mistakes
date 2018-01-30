---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: Cloudera Manager - Duplicate entry 'zookeeper' for key 'NAME'.
category: ['Linux', 'Centos', 'Redhat', 'Cloudera', 'Kafka', 'Zookeeper', 'Cluster']
tags: ['linux', 'centos', 'redhat', 'cloudera', 'kafka', 'zookeeper', 'cluster']
---

We had recently built a cluster using cloudera API's and had all the services running on it with Kerberos enabled.
Next we had a requirement to add another kafka cluster to our already exsisting cluster in cloudera manager. Since it is a quick task to get the `zookeeper` and `kafka` up and running.
We decided to get this done using the cloudera manager instead of the API's. But we faced the `Duplicate entry 'zookeeper' for key 'NAME'` issue as described in the bug below.
 
[`https://issues.cloudera.org/browse/DISTRO-790`](https://issues.cloudera.org/browse/DISTRO-790)

    I have set up two clusters that share a Cloudera Manger.
    The first I set up with the API and created the services with capital letter names, e.g., ZOOKEEPER, HDFS, HIVE.
    Now, I add the second cluster using the Wizard.
    
    Add Cluster->Select Hosts->Distribute Parcels->Select base HDFS Cluster install
    
    On the next page i get SQL errros telling that the services i want to add already exist. 
    I suspect that the check for existing service names does not include capitalized letters.
    Note that renaming the services does not help, as it would only change the DisplayName in the database and not the Name column, 
    which is unfortunately also a key column. 

    Here the excerpt of the error message (attached full log).
    javax.persistence.PersistenceException:org.hibernate.exception.ConstraintViolationException: could not perform addBatch
    
    at AbstractEntityManagerImpl.java line 1387
    in org.hibernate.ejb.AbstractEntityManagerImpl convert()
    Caused by: java.sql.BatchUpdateException:Duplicate entry 'zookeeper' for key 'NAME'
    
    at PreparedStatement.java line 2024
    in com.mysql.jdbc.PreparedStatement executeBatchSerially()

Solutions: 

    For now solution was to use the API and change the names to other than what we had used earlier. 
    Example we used `ZOOKEEPER` in out earlier build, so we changed it to `ZOOKEEPER001` in the new build. 
    
    NOTE: Apparently Cloudera manager does not look for CAPITAL `ZOOKEEPER`, 
    so if it did, then CM would generated a different name automatically.

And yes this bug was fixed recently and might take a couple of versions to see it on the stable release.
Issue is that when we deploy our services using API we named them as `ZOOKEEPER`  (all caps) but cloudera manager check for all versions except ‘Capital’.
so it continues to build and  fail with Duplicate error. If it detects then it would create a different name automatically.
Since this was not working, current workaround is to deploy the services using Cloudera API using a different name (Currently named as ZOOKEEPER001/KAFKA001) . 

Another fix would be to change the API script to change the service name to `SERVICE_NAME_<first 3 characters of CLUSTER_NAME>`, example: ‘ZOOKEEPER_HAD’, ‘ZOOKEEPER_KAF’ or a number `ZOOKEEPER_1`.


    