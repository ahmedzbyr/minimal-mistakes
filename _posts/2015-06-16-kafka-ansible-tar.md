---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: Ansible Playbook - Setup Kafka Cluster.  
category: ['Hadoop', 'Linux', 'Kafka', 'Ansible']
tags: ['linux', 'ansible', 'kafka', 'hadoop', 'zookeeper']
---


This is a simple Kafka setup. In this setup we are running `kafka` over a dedicated `zookeeper` service. 
(NOT the standalone zookeeper which comes with `kafka`)

Before we start read more information about Zookeeper/Kafka in the below link.

1. Setup Zookeeper. 
2. Setup Kafka. Server running on ports 9091/9092 ports on each server.

##  Before we start.

Download [`kafka_2.9.2-0.8.2.1.tgz`](http://mirror.metrocast.net/apache/kafka/0.8.2.1/kafka_2.9.2-0.8.2.1.tgz) to `file_archives` directory.

Download [`zookeeper-3.4.5-cdh5.1.2.tar.gz`](http://archive.cloudera.com/cdh5/cdh/5/zookeeper-3.4.5-cdh5.1.2.tar.gz) to `file_archives` directory.

Download [`jdk-7u75-linux-x64.tar.gz`](http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html# jdk-7u75-oth-JPR) to `file_archives` directory.

##  Get the script from Github.

Below is the command to clone. 

    ahmed@ahmed-server ~]$ git clone https://github.com/zubayr/ansible_kafka_tarball


##  Step 1: Update Hosts File.

Update the host file to reflect your server IPs.
Currently `hosts` file looks as below.

    [zookeepernodes]
    10.10.18.10 zookeeper_id=1
    10.10.18.12 zookeeper_id=2
    10.10.18.13 zookeeper_id=3
    
    [kafkanodes]
    10.10.18.10 kafka_broker_id1=11 kafka_port1=9091 kafka_broker_id2=12 kafka_port2=9092
    10.10.18.12 kafka_broker_id1=13 kafka_port1=9091 kafka_broker_id2=14 kafka_port2=9092
    
##  Step 2: Update `group_vars` information as required.

Update users/password and Directory information in `group_vars/all` file.
Currently we have the below information.
    
    #  --------------------------------------
    #  USERs
    #  --------------------------------------
    
    #  password below for all the users is `hdadmin@123`
    
    zookeeper_user: zkadmin
    zookeeper_group: zkadmin
    zookeeper_password: <encrypted_password_here>
    
    kafka_user: kafkaadmin
    kafka_group: kafkaadmin
    kafka_password: <encrypted_password_here>
    
    
    #  --------------------------------------
    #  COMMON FOR INSTALL PATH
    #  --------------------------------------
    
    #  Common Location information.
    common:
      install_base_path: /usr/local
      soft_link_base_path: /opt

      
Password can be generated using the below python snippet.

    #  Password Generated using python command below.
    python -c "from passlib.hash import sha512_crypt; \
                            import getpass; print sha512_crypt.encrypt(getpass.getpass())"

Here is the execution. After entering the password you will get the encrypted password which can be used in the user creation.

    ahmed@ahmed-server ~]$ python -c "from passlib.hash \
                            import sha512_crypt; import getpass; \
                            print sha512_crypt.encrypt(getpass.getpass())"
    Enter Password: *******
    $6$rounds=40000$/VINUa2uPsmGK/2xnmOt80TjDwbof9rNvnYY6icCkdAR2qrFquirBtT1
    ahmed@ahmed-server ~]$      
      
##  Step 3: Update `default` information in `roles/<install_role>/default/main.yml`.

Update the `default` values if required.


##  Step 4: Executing.

Below is the command. 
    
    ahmed@ahmed-server ansible_kafka_tarball]$ ansible-playbook ansible_kafka.yml -i hosts --ask-pass