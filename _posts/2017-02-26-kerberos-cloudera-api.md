---
title: Enable Kerberos Using Cloudera API.
category: ['Linux', 'Cloudera', 'Hadoop', 'Cloudera-Api', 'Kerberos']
tags: ['linux', 'cloudera', 'hadoop', 'cloudera-api', 'kerberos']
---

Python API for cloudera is really nice, apart from getting the cluster setup, we can also do configuration and automation. We use a lot of automation using Chef/Ansible, but cloudera API give more control over the cluster.

One of the awesome features of cloudera API is to setup kerberos for the cluster, which otherwise done manually (command line) is very tricky task (loads of this to go wrong).

For our kerberos setup we will need to complete below steps. 

1. Cloudera Manager Configuration.
2. Stop Cluster.
3. Stop Cloudera Manager service. 
4. Service configuration for all the services which needs updates like `HDFS`, `HBASE`, `KAFKA`, `ZOOKEEPER`, `HUE`.
5. Creating `KT_RENEWER` for HUE service. 
6. Start Cloudera Manager Services.
7. Start Cluster.


## Steps To `kerberize` Cloudera Cluster

Step 1: Configure cloudera with LDAP information.

``` python
#
# Deploy LDAP configuration for CM
#
self.deploy_cm_ldap_configuration()
```

Step 2: Import Admin credentials, using the username / password, which has permission to create/delete AD service accounts.

``` python 
#
# Creating Admin Credentials
#
self.cm_import_admin_credentials()
```

Step 3: Stopping cloudera cluster services.

``` python
#
# Stopping Cluster
#
self.gets_cm_cluster.stop().wait()
logging.debug("Waiting for CLUSTER to stop completely !!!")
time.sleep(5)
```

Step 4: Stopping cloudera management services.

``` python
#
# Stopping CM services
#
self.get_mgmt_services.stop().wait()
logging.debug("Waiting for CM to stop completely !!!")
time.sleep(5)
```

Step 5: Update Zookeeper configuration.

``` python
#
# Deploy Kerberos Config Zookeeper
#
logging.debug("Deploy Service Configuration!!!")
zk_service = Zookeeper(self.cluster, self.config, self.cloudera_manager)
zk_service.update_configuration()
logging.debug("Deploy Service Configuration COMPLETE!!!")
```

Step 6: Update HDFS configuration.

``` python
#
# Deploy Kerberos Config Hdfs
#
logging.debug("Deploy Service Configuration!!!")
hdfs_service = Hdfs(self.cluster, self.config, self.cloudera_manager)
hdfs_service.update_configuration()
logging.debug("Deploy Service Configuration COMPLETE!!!")
```

Step 6: Update HBASE configuration.

``` python
#
# Deploy Kerberos Config Hbase
#
logging.debug("Deploy Service Configuration!!!")
hbase_service = Hbase(self.cluster, self.config, self.cloudera_manager)
hbase_service.update_configuration()
logging.debug("Deploy Service Configuration COMPLETE!!!")
```

Step 7: Update KAFKA configuration.

``` python
#
# Deploy Kerberos Config Kafka
#
logging.debug("Deploy Service Configuration!!!")
kafka_service = Kafka(self.cluster, self.config, self.cloudera_manager)
kafka_service.update_configuration()
logging.debug("Deploy Service Configuration COMPLETE!!!")
```

Step 8: Update HUE configuration.

``` python
#
# Deploy Kerberos Config Hue
#
logging.debug("Deploy Service Configuration!!!")
hue_service = Hue(self.cluster, self.config, self.cloudera_manager)
hue_service.update_configuration()
hue_service.add_service_kt_renewer_to_cluster()
logging.debug("Deploy Service Configuration COMPLETE!!!")
```

Step 9: Generating Credentials.

``` python
#
# Generated kerberos credentials in AD.
#
self.cm_generate_credentials()
time.sleep(5)
```

Step 10: Deploy Client Configuration.

``` python
#
# Deploy Client Configuration
#
logging.info("Deploying Client Config...")
self.deploy_client_configuration()
logging.debug("Waiting for CLUSTER to deploy completely !!!")
time.sleep(5)
```

Step 11: Starting CM Services.

``` python
#
# Starting CM services.
#
self.get_mgmt_services.start().wait()
logging.debug("Waiting for CM to start completely !!!")
time.sleep(5)
```

Step 12: Starting cluster services.

``` python
#
# Restart Cluster based on stale configuration and redeploy config if required.
#
self.gets_cm_cluster.start().wait()
logging.info("Cluster Kerberos Deployment Complete.")
```

## Important configuration information. 

**LDAP setup.**

	'cm_ldap': {
	    'KDC_HOST': 'adserver1.service.ahmedinc.com',
	    'SECURITY_REALM': 'SERVICE.AHMEDINC.COM',
	    'KRB_ENC_TYPES': 'rc4-hmac',
	    'KDC_ACCOUNT_CREATION_HOST_OVERRIDE': 'adserver2.service.ahmedinc.com',
	    'AD_KDC_DOMAIN': 'OU=accounts,OU=test-lab-ou,OU=bigdata,DC=service,DC=ahmedinc,DC=com',
	    'AD_DELETE_ON_REGENERATE': True,
	    'AD_ACCOUNT_PREFIX': 'Lab1Test'
	  }

**LDAP credentials for user with `create`, `delete`, `modify` permissions.**

	'cm_kdc_import_credentials': {
		'kdc_password': 'Ahm3d@123',
		'kdc_username': 'service-acc-lab@SERVICE.AHMEDINC.COM'
	}


**Configuration to connect to Cloudera Manager.**

	'cm': {
	    'username': 'admin',
	    'tls': False,
	    'host': 'server-admin-node.ahmedinc.com',
	    'api-version': 13,
	    'password': 'admin',
	    'port': 7180
	}


**Cluster configuration.**

	'cluster': {
	    'version': 'CDH5',
	    'hosts': [
	      'server-admin-node.ahmedinc.com',
	      'server-edge-node.ahmedinc.com',
	      'server-worker-node.ahmedinc.com'
	    ],
	    'name': 'AutomatedHadoopCluster',
	    'fullVersion': '5.8.3'
	}

**Service Configuration Hdfs.**

    'HDFS': {
      'config': {
        'hadoop_security_authentication': 'kerberos',
        'trusted_realms': 'USERACC.AHMEDINC.COM',
        'hadoop_security_authorization': True
      },
      'roles': [
        {
          'config': {
            'dfs_datanode_http_port': 1006,
            'dfs_datanode_data_dir_perm': 700,
            'dfs_datanode_port': 1004
          },
          'hosts': [
            'server-admin-node.ahmedinc.com',
            'server-edge-node.ahmedinc.com',
            'server-worker-node.ahmedinc.com'
          ],
          'group': 'DATANODE'
        }
      ]
    }

**Service Configuration Hbase.**

    'HBASE': {
      'config': {
        'hbase_thriftserver_security_authentication': 'auth',
        'hbase_security_authorization': True,
        'hbase_security_authentication': 'kerberos'
      }
    }


**Service Configuration Zookeeper.**

    'ZOOKEEPER': {
      'config': {
        'enableSecurity': True
      }
    }

**Service Configuration Kafka.**


    'KAFKA': {
      'config': {
        'kerberos.auth.enable': True
      }
    }

**Service Configuration Hue.**

	'HUE': {
      'roles': [
        {
          'group': 'KT_RENEWER',
          'hosts': [
            'server-admin-node.ahmedinc.com'
          ]
        }
      ]
    }


NOTE :  In a `kerberos` setup, when we do a `update_config`, `generate_missing_credentials` command is triggered.

- we need to wait for this command to complete, before we start the next `update_config` or else `generate_credential` command will FAIL.
- `get_commands` will return a List of `ApiCommand` responses.  we iterate through all the command to complete. [currently we will receive only one command]
- [`get_commands`](https://cloudera.github.io/cm_api/epydoc/5.10.0/cm_api.endpoints.cms.ClouderaManager-class.html#get_commands)
- [`ApiCommand`](https://cloudera.github.io/cm_api/apidocs/v15/ns0_apiCommand.html)

Code snippet.

``` python
current_running_commands = self.cm.get_commands()
logging.debug("Current Running Commands: " + str(current_running_commands))
if not current_running_commands:
    logging.info("Currently no command is running.")
elif current_running_commands:
    for cmd in current_running_commands:
        if cmd.wait().success:
            logging.info("Command Completed. moving on..")
else:
    logging.error("We are in ELSE, something went wrong. :(")
    sys.exit(1)

logging.info("Configuration Updated Successfully..")
time.sleep(5)
```

**Update Configuration for Cloudera Manager LDAP.**

``` python
def deploy_cm_ldap_configuration(self):
	"""
	    Updating LDAP configuratio no CM server.
	    This will require a `sudo servive cloudera-scm-server restart`, we are doing a restart in the end of the script.
	:return:
	"""
	cm_api_handle = ApiResource(config['cm']['host'],
	                                      config['cm']['port'],
	                                      config['cm']['username'],
	                                      config['cm']['password'],
	                                      config['cm']['tls'],
	                                      version=config['cm']['api-version'])
	cluster = cm_api_handle.get_cluster(config['cluster']['name'])
	cloudera_manager = cm_api_handle.get_cloudera_manager()
	cloudera_manager.update_config(self.config['cm_ldap'])
```

**Example Configuration update for Zookeeper.**

``` python
def update_configuration(self):
	"""
	    Update service configurations
	:return:
	"""
	cm_api_handle = ApiResource(config['cm']['host'],
	                                      config['cm']['port'],
	                                      config['cm']['username'],
	                                      config['cm']['password'],
	                                      config['cm']['tls'],
	                                      version=config['cm']['api-version'])
	cluster = cm_api_handle.get_cluster(config['cluster']['name'])
	get_service = cluster.get_service('ZOOKEEPER')
	get_service.update_config(config['ZOOKEEPER']['config'])
	logging.info("Service Configuration Updated.")
	
	#   In a kerberos setup, when we do a `update_config`
	#      `generate_missing_credentials` command is triggered.
	#      we need to wait for this command to complete,
	#      before we start the next `update_config`
	#      or else `generate_credential` command will FAIL.
	
	cm = cm_api_handle.get_cloudera_manager()
	current_running_commands = cm.get_commands()
	logging.debug("Current Running Commands: " + str(current_running_commands))
	if not current_running_commands:
	    logging.info("Currently no command is running.")
	elif current_running_commands:
	    for cmd in current_running_commands:
	        if cmd.wait().success:
	            logging.info("Command Completed. moving on..")
	else:
	    logging.error("We are in ELSE, something went wrong. :(")
	    sys.exit(1)
	
	logging.info("Configuration Updated Successfully..")
	time.sleep(5)
```

**Creating `KT_RENEWER` code snippet.**

Code snippet for `kt_renewer`

``` python
def create_kt_renewer_service(self):
	
	
	cm_api_handle = ApiResource(config['cm']['host'],
	                                      config['cm']['port'],
	                                      config['cm']['username'],
	                                      config['cm']['password'],
	                                      config['cm']['tls'],
	                                      version=config['cm']['api-version'])
	cluster = cm_api_handle.get_cluster(config['cluster']['name'])
	get_service = cluster.get_service('HUE')
	
	
	for role in self.config['HUE']['roles']:
	    role_group = self.get_service.get_role_config_group("{0}-{1}-BASE".format('HUE', role['group']))
	    # Update the group's configuration.
	    # [https://cloudera.github.io/cm_api/epydoc/5.10.0/cm_api.endpoints.role_config_groups.ApiRoleConfigGroup-class.html#update_config]
	    role_group.update_config(role.get('config', {}))
	    self.create_roles(get_service, role, role['group'])
```

Creating Roles.

``` python
def create_roles(self, service, role, group):
	"""
	Create individual roles for all the hosts under a specific role group
	
	:param role: Role configuration from yaml
	:param group: Role group name
	"""
	role_id = 0
	for host in role.get('hosts', []):
	    role_id += 1
	    role_name = '{0}-{1}-{2}'.format('HUE', group, role_id)
	    logging.info("Creating Role name as: " + str(role_name))
	    try:
	        service.get_role(role_name)
	    except ApiException:
	        service.create_role(role_name, group, host)
```

We are done.


### Useful Links.

- [Update Config](https://cloudera.github.io/cm_api/epydoc/5.10.0/cm_api.endpoints.role_config_groups.ApiRoleConfigGroup-class.html#update_config)
- [Get Command](https://cloudera.github.io/cm_api/epydoc/5.10.0/cm_api.endpoints.cms.ClouderaManager-class.html#get_commands)
- [Api Command](https://cloudera.github.io/cm_api/apidocs/v15/ns0_apiCommand.html)