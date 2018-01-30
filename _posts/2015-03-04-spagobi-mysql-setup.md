---
title: Installing SpagoBI 5.1 on Centos 6.5 -Tomcat 7 with MySQL 5.6.
category: ['Linux', 'Spagobi', 'Bi']
tags: ['linux', 'spagobi', 'bi', 'spago', 'centos', 'tomcat', 'mysql']
---

The SpagoBI project is a free software/open source initiative by the SpagoBI Labs of Engineering Group. It aims to realize the most complete 100% open source business intelligence suite, aggregating developers, integrators, companies, users and passionate people in an open community.

<http://www.spagobi.org/homepage/opensource/faqs/>

##  Prerequiste Downloads and setup.

1. Install [JAVA JDK.](http://download.oracle.com/otn-pub/java/jdk/7u75-b13/jdk-7u75-linux-x64.rpm)
2. Install MySQL on CentOS 6.5, HOWTO Install Instructions are [here](http://sharadchhetri.com/2013/12/26/install-mysql-server-5-6-in-centos-6-x-and-red-hat-6-x-linux/).
3. Download [SpagoBI 5.1.](http://download.forge.ow2.org/spagobi/All-In-One-SpagoBI-5.1-21012015.zip)
4. Download [Mysql Script](http://download.forge.ow2.org/spagobi/mysql-dbscript-5.1.0_19012015.zip) from SpagoBI Website.
5. Download [Mysql Connector.](http://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.34.tar.gz)
6. Download [Tomcat 7.](http://apache.bytenet.in/tomcat/tomcat-7/v7.0.59/bin/apache-tomcat-7.0.59.tar.gz) (**Optional** - if you want to run SpagoBI on a Dedicated Tomcat Server.)



##  Extracting SpagoBI.

Extract `All-In-One-SpagoBI-5.1-21012015.zip`

	[ahmed@ahmed-server ~]# unzip All-In-One-SpagoBI-5.1-21012015.zip

Move to `/opt` and create a `softlink` as `spagobi`

	[ahmed@ahmed-server ~]# mv All-In-One-SpagoBI-5.1-21012015 /opt/
	[ahmed@ahmed-server ~]# ln -s  /opt/All-In-One-SpagoBI-5.1-21012015 /opt/spagobi

Setting Heap memory in `setenv.sh`
	
	[ahmed@ahmed-server ~]# vim /opt/spagobi/bin/setenv.sh

Add below content. Depending on the Server Memory set the `Xms` and `Xmx` appropriately. 

	export JAVA_OPTS="$JAVA_OPTS\
	  -server\
	  -Xms8g\
	  -Xmx16g\
	  -XX:MaxPermSize=512m\
	  -XX:MaxNewSize=512m\
	  -XX:NewSize=512m\
	  -XX:SurvivorRatio=12"
	
	

##  Update SpagoBI `lib` with Mysql Connector. 

Extract the connector.

	[ahmed@ahmed-server ~]# tar xvzf mysql-connector-java-5.1.34.tar.gz

Copy `jar` to `spago` lib directory.

	[ahmed@ahmed-server ~]# cp mysql-connector-java-5.1.34/mysql-connector-java-5.1.34-bin.jar \
                                                                                /opt/spagobi/lib/

NOTE : If you already have a older connector in the directory `/opt/spagobi/lib/` then remove it. USE THE LATEST CONNECTOR.

##  Setting up Mysql Database.

Before we populated the scripts we need to create basic setup on Mysql.

###  Setting `spagobi` Database and `spagobi` User.

Assuming that `mysql-server` is installed and we have set the `root` password as `root@123`.

	[ahmed@ahmed-server ~]# mysql -uroot -proot@123
	Warning: Using a password on the command line interface can be insecure.
	Welcome to the MySQL monitor.  Commands end with ; or \g.
	Your MySQL connection id is 475739
	Server version: 5.6.21-log MySQL Community Server (GPL)
	
	Copyright (c) 2000, 2014, Oracle and/or its affiliates. All rights reserved.
	
	Oracle is a registered trademark of Oracle Corporation and/or its
	affiliates. Other names may be trademarks of their respective
	owners.
	
	Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

Creating `spagobi` database.
	
	mysql> create database spagobi;
	Query OK, 0 rows affected (0.00 sec)
	
Creating a user `spagobi` with password as `spagobi` for `spagobi` database.

	mysql> grant all privileges on spagobi.* to spagobi@localhost identified by 'spagobi';
	Query OK, 0 rows affected (0.00 sec)

**Optional** - If we are running mysql in a different machine then we have give permission to the server which is trying to connect to `mysql-server`. Assuming that web server is `10.10.10.18` we give the permission as below.

	mysql> grant all privileges on spagobi.* to spagobi@10.10.10.18 identified by 'spagobi';
	Query OK, 0 rows affected (0.00 sec)

Commit and we are done.
	
	mysql> commit;
	Query OK, 0 rows affected (0.00 sec)
	
	mysql> exit


### Populating Tables for SpagoBI.

Extract `script` from `mysql-dbscript-5.1.0_19012015.zip`

	[ahmed@ahmed-server ~]# mkdir mysql-spagobi
	[ahmed@ahmed-server ~]# cd mysql-spagobi
	[ahmed@ahmed-server mysql-spagobi]# unzip mysql-dbscript-5.1.0_19012015.zip 
	[ahmed@ahmed-server mysql-spagobi]# ls -l
	total 136
	-rw-r--r-- 1 root root   4400 Nov 19 12:38 MySQL_create_quartz_schema.sql
	-rw-r--r-- 1 root root   7954 Dec  3 11:44 MySQL_create_social.sql
	-rw-r--r-- 1 root root 111910 Nov 19 12:38 MySQL_create.sql
	-rw-r--r-- 1 root root    856 Nov 19 12:38 MySQL_drop_quartz_tables.sql
	-rw-r--r-- 1 root root   3821 Nov 19 12:38 MySQL_drop.sql
	[ahmed@ahmed-server mysql-spagobi]#	


Now lets populate tables.
 
	[ahmed@ahmed-server ~]# mysql -uspagobi -pspagobi spagobi < MySQL_create.sql
	[ahmed@ahmed-server ~]# mysql -uspagobi -pspagobi spagobi < MySQL_create_quartz_schema.sql
	[ahmed@ahmed-server ~]# mysql -uspagobi -pspagobi spagobi < MySQL_create_social.sql
 
Now we are ready.

##  SpagoBI MySQL Connection Setup.

### Update `server.xml`.

	/opt/spagobi/conf/server.xml

Change the `GlobalNamingResources` to the below tag.

	<!-- server.xml -->
	<GlobalNamingResources> 
		<!-- Editable user database that can also be used by UserDatabaseRealm to authenticate users--> 
		<Resource auth="Container" 
                        description="User database that can be updated and saved" 
                        factory="org.apache.catalina.users.MemoryUserDatabaseFactory" 
                        name="UserDatabase" 
                        pathname="conf/tomcat-users.xml" 
                        type="org.apache.catalina.UserDatabase"/> 
                  
		<Environment name="spagobi_resource_path" 
                        type="java.lang.String" 
                        value="${catalina.base}/resources"/> 
                     
		<Environment name="spagobi_sso_class" 
                        type="java.lang.String" 
                        value="it.eng.spagobi.services.common.FakeSsoService"/> 
                        
		<Environment name="spagobi_service_url" 
                        type="java.lang.String" 
                        value="http://localhost:8080/SpagoBI"/> 
                        
		<Environment name="spagobi_host_url" 
                        type="java.lang.String" 
                        value="http://localhost:8080"/> 
                        
		<Resource auth="Container" 
                        factory="de.myfoo.commonj.work.FooWorkManagerFactory" 
                        maxThreads="5" 
                        name="wm/SpagoWorkManager" 
                        type="commonj.work.WorkManager"/> 
                        
		<Resource auth="Container" 
                        driverClassName="com.mysql.jdbc.Driver" 
                        maxActive="20" maxIdle="10" maxWait="-1" 
                        name="jdbc/spagobi" password="spagobi" 
                        type="javax.sql.DataSource" 
                        url="jdbc:mysql://10.10.10.88:3306/spagobi" 
                        username="spagobi"/>  
                        
	</GlobalNamingResources> 


### Update `context.xml`.

	/opt/spagobi/webapps/SpagoBI/META-INF/context.xml

Change the contents of the file as below.

	<Context docBase="SpagoBIProject" path="/SpagoBI" privileged="true" 
                            reloadable="true" source="org.eclipse.jst.j2ee.server:SpagoBIProject">  
		<ResourceLink global="jdbc/spagobi" 
                                name="jdbc/spagobi" type="javax.sql.DataSource"/> 
		<ResourceLink global="spagobi_resource_path" 
                                name="spagobi_resource_path" type="java.lang.String"/> 
		<ResourceLink global="spagobi_sso_class" 
                                name="spagobi_sso_class" type="java.lang.String"/> 
		<ResourceLink global="spagobi_host_url" 
                                name="spagobi_host_url" type="java.lang.String"/>  
	</Context> 

### Update `hibernate.cfg.xml`.

	/opt/spagobi/webapps/SpagoBI/WEB-INF/classes/hibernate.cfg.xml

Update the `hibernate-configuration` tag, leave the other `tag` details as it is.

	<hibernate-configuration> 
		<session-factory name="HibernateSessionFactoryMySQL"> 
		<property name="hibernate.connection.datasource">java:/comp/env/jdbc/spagobi</property> 
		<property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>
		<property name="hibernate.cache.use_second_level_cache">false</property> 
		<property name="hibernate.cache.use_query_cache">false</property> 
		<!-- 
			Leave rest of the Configuration as it is.
		--> 
		</session-factory> 
	</hibernate-configuration> 

### Update `jbpm.hibernate.cfg.xml`.

	/opt/spagobi/webapps/SpagoBI/WEB-INF/classes/jbpm.hibernate.cfg.xml

Update the `hibernate-configuration` tag, leave the other `tag` details as it is.

	<hibernate-configuration> 
		<session-factory> 
			<property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>
			<property name="hibernate.connection.datasource">java:comp/env/jdbc/spagobi</property> 
			<property name="hibernate.cache.use_second_level_cache">false</property> 
			<property name="hibernate.cache.use_query_cache">false</property> 
			<!-- 
				Leave rest of the Configuration as it is.
			--> 
		</session-factory> 
	</hibernate-configuration> 

###  Update `quartz.properties`.

	/opt/spagobi/webapps/SpagoBI/WEB-INF/classes/quartz.properties

Comment below
 
	org.quartz.jobStore.driverDelegateClass=org.quartz.impl.jdbcjobstore.HSQLDBDelegate 

Uncomment Below for Mysql.

	org.quartz.jobStore.driverDelegateClass=org.quartz.impl.jdbcjobstore.StdJDBCDelegate 


##  Starting SpagoBI Server.

Lets go to Location and start.

	[ahmed@ahmed-server ~]# cd /opt/spagobi/bin

Since we are not using HSQLDB and have configured MySQL, we need to start just the `tomcat` server.

	[ahmed@ahmed-server bin]# ./startup.sh 

This will start the server on 8080.

!["SPAGO BI"](https://lh6.googleusercontent.com/-2k0lkxw6NfU/VPcOwEScBZI/AAAAAAAAks4/tokQIq0vq3k/w621-h440-no/SPAGOBI.PNG "SPAGO BI")


##  Moving SpagoBI to Dedicated `Tomcat7` Server.

NOTE : Before we start stop `spagobi` server, which we started earlier.

	[ahmed@ahmed-server ~]# /opt/spagobi/bin/shutdown.sh

Extract tomcat to `/opt` and create a `softlink` as `tomcat`.

	[ahmed@ahmed-server ~]# tar xvzf apache-tomcat-7.0.59.tar.gz -C /opt
	[ahmed@ahmed-server ~]# ln -s /opt/apache-tomcat-7.0.59 /opt/tomcat


###  Copy `lib` Files.

Copy without overwrite.

	[ahmed@ahmed-server ~]# cp -n /opt/spagobi/lib/* /opt/tomcat/lib/
   
Option Details

	-n, --no-clobber
          do  not  overwrite  an  existing  file  (overrides a previous -i option)

###  Copy webapps Directory.

Remove existing `webapps` directory as we dont need it any more.

	[ahmed@ahmed-server ~]# rm -rf /opt/tomcat/webapps

Since we have already updated all the config files in the `/opt/spagobi/webapps` directory, we will copy them as it is.
	
	[ahmed@ahmed-server ~]# cp -f /opt/spagobi/webapps /opt/tomcat/

###  Copy conf `server.xml` from /opt/spago.

Lets copy `server.xml` and overwrite the file.

	[ahmed@ahmed-server ~]# cp -f /opt/spagobi/conf/server.xml /opt/tomcat/conf/server.xml


###  Copy `setenv.sh` to `/opt/tomcat/bin`.

Finally copy `setenv.sh` file to `bin` directory.

	[ahmed@ahmed-server ~]# cp  /opt/spagobi/bin/setenv.sh  /opt/tomcat/bin/


###  Starting Tomcat Server.


	[ahmed@ahmed-server ~]# sh /opt/tomcat/bin/startup.sh

We are done.
