---
title: Installing MongoDB on Ubuntu 14 LTS.
category: ['Linux', 'Mongodb', 'Ubuntu']
tags: ['linux', 'mongo', 'mongodb', 'ubuntu', 'nagios']
---

MongoDB is an open-source document database, and leading NoSQL database. MongoDB is written in c++. Below is a brief document about installing a `mongodb` on a test node to try it out.

### Import the public key used by the package management system.

Signed Packages for `dpkg` and `apt`

``` ruby
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927

```
Output.

``` ruby
ahmed@ubuntu:~$ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927
[sudo] password for ahmed:
Executing: gpg --ignore-time-conflict --no-options --no-default-keyring --homedir /tmp/tmp.ApILz9KbVd --no-auto-check-trustdb --trust-model always --keyring /etc/apt/trusted.gpg --primary-keyring /etc/apt/trusted.gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927
gpg: requesting key EA312927 from hkp server keyserver.ubuntu.com
gpg: key EA312927: public key "MongoDB 3.2 Release Signing Key <packaging@mongodb.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)
```

### Create Repo

Create the /etc/apt/sources.list.d/mongodb-org-3.2.list list file using the command appropriate for your version of Ubuntu:

``` ruby
echo "deb http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list
```

Output.

``` ruby
ahmed@ubuntu:~$ echo "deb http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list
deb http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.2 multiverse
```

### Reload local package database.


``` ruby
sudo apt-get update
```

### Install MongoDB


``` ruby
sudo apt-get install -y mongodb-org
```


### Start MongoDB.

Issue the following command to start mongod:

``` ruby
sudo service mongod start
```

### Verify that MongoDB has started successfully

Verify that the mongod process has started successfully by checking the contents of the log file at ``/var/log/mongodb/mongod.log` for a line reading

``` ruby
[initandlisten] waiting for connections on port <port>
```

where ``<port>`` is the port configured in ``/etc/mongod.conf`, `27017` by default.

Output

``` ruby
ahmed@ubuntu:~$ sudo tail -f /var/log/mongodb/mongod.log
2016-09-14T17:44:54.437-0700 I CONTROL  [initandlisten]
2016-09-14T17:44:54.437-0700 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/enabled is 'always'.
2016-09-14T17:44:54.437-0700 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2016-09-14T17:44:54.437-0700 I CONTROL  [initandlisten]
2016-09-14T17:44:54.437-0700 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/defrag is 'always'.
2016-09-14T17:44:54.437-0700 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2016-09-14T17:44:54.437-0700 I CONTROL  [initandlisten]
2016-09-14T17:44:54.439-0700 I FTDC     [initandlisten] Initializing full-time diagnostic data capture with directory '/var/lib/mongodb/diagnostic.data'
2016-09-14T17:44:54.439-0700 I NETWORK  [HostnameCanonicalizationWorker] Starting hostname canonicalization worker
2016-09-14T17:44:54.533-0700 I NETWORK  [initandlisten] waiting for connections on port 27017
```

### Importing First Dataset using `mongoimport`.

Get the file from link below.

```
wget https://github.com/zubayr/big_data_learning/blob/master/bigData/mongodb/dataset/companies.zip
unzip companies.zip
```

Output.

``` ruby
ahmed@ubuntu:~$ wget https://github.com/zubayr/big_data_learning/raw/master/bigData/mongodb/dataset/companies.zip
--2016-09-14 17:51:12--  https://github.com/zubayr/big_data_learning/raw/master/bigData/mongodb/dataset/companies.zip
Resolving github.com (github.com)... 192.30.253.112
Connecting to github.com (github.com)|192.30.253.112|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://raw.githubusercontent.com/zubayr/big_data_learning/master/bigData/mongodb/dataset/companies.zip [following]
--2016-09-14 17:51:28--  https://raw.githubusercontent.com/zubayr/big_data_learning/master/bigData/mongodb/dataset/companies.zip
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 151.101.100.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|151.101.100.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 15493946 (15M) [application/octet-stream]
Saving to: ‘companies.zip.1’

100%[=======================================>] 15,493,946   590KB/s   in 34s   

ahmed@ubuntu:~$ unzip companies.zip
Archive:  companies.zip
  inflating: companies.json          
ahmed@ubuntu:~$ ls
companies.json  Desktop    Downloads         Music     Public     Videos
companies.zip   Documents  examples.desktop  Pictures  Templates
ahmed@ubuntu:~$

```


#### Importing dataset.

``` ruby
mongoimport --db company --collection companies --file companies.json
```
Output. `mongoimport` will by default connect to `localhost` on port `27017`, if we are trying to import to a mongodb on a different machine, then need to pass the `--host` and `--port` options.

``` ruby
ahmed@ubuntu:~$ mongoimport --db company --collection companies --file companies.json
2016-09-14T17:54:34.032-0700	connected to: localhost
2016-09-14T17:54:37.025-0700	[#########...............] company.companies	30.0MB/74.6MB (40.3%)
2016-09-14T17:54:40.033-0700	[###################.....] company.companies	61.8MB/74.6MB (82.8%)
2016-09-14T17:54:41.274-0700	[########################] company.companies	74.6MB/74.6MB (100.0%)
2016-09-14T17:54:41.274-0700	imported 18801 documents
ahmed@ubuntu:~$
```


### Setting up Authentication.

#### Create the user administrator.

``` ruby
use admin
db.createUser(
  {
    user: "mongoadmin",
    pwd: "ahmed@123",
    roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
  }
)

```
Output.

``` ruby
> use admin
> db.createUser({user:"mongoadmin",pwd:"ahmed@123",roles:[{role:"userAdminAnyDatabase",db:"admin"}]})
Successfully added user: {
	"user" : "mongoadmin",
	"roles" : [
		{
			"role" : "userAdminAnyDatabase",
			"db" : "admin"
		}
	]
}
```

#### Re-start the MongoDB instance with access control.

Re-start the mongod instance with the `--auth` command line option or, if using a configuration file, the `security.authorization` setting.

``` ruby
mongod --auth --port 27017 --dbpath /data/db1
```
Or Update the configuration `/etc/mongod.conf` file with below info.

``` ruby
security:
  authorization: enabled
```

#### To authenticate during connection.

``` ruby
mongo --port 27017 -u "mongoadmin" -p "ahmed@123" --authenticationDatabase "admin"
```

#### Create additional users as needed for your deployment.

``` ruby
use company
db.createUser(
  {
    user: "ahmed",
    pwd: "ahmed@123",
    roles: [ { role: "readWrite", db: "company" },
             { role: "read", db: "test" } ]
  }
)
```

Connect and authenticate as `ahmed`.

``` ruby
mongo --port 27017 -u "ahmed" -p "ahmed@123" --authenticationDatabase "company"
```

#### Insert into a collection as `ahmed`.

``` ruby
> use company
> db.authtesting.insert({x:1,y:1})
WriteResult({ "nInserted" : 1 })
> db.authtesting.findOne()
{ "_id" : ObjectId("57d9f85a3d1dcdf58c16cab3"), "x" : 1, "y" : 1 }
>
```

### Bibliography.

Issue getting monitoring data in `nagios`.

#### Executing command from the `nagios` server.

``` ruby
[ahmed@localhost libexec]$ ./check_mongodb_2.py -H 192.168.94.138 -P 27017 -u admin -p admin1 -A databases -W 5 -C 10
CRITICAL - General MongoDB Error: command SON([('authenticate', 1), ('user', u'admin'), ('nonce', u'42110dc29ee7fe6b'), ('key', u'827a2b0e4af97e88560800ab86b04e57')]) failed: auth failed
```

#### On the mongodb server.

``` ruby
2016-09-14T19:11:12.142-0700 I ACCESS   [conn114] Successfully  authenticated as principal admin on admin
2016-09-14T19:11:32.892-0700 I NETWORK  [initandlisten] connection accepted from  192.168.94.130:48657 #115 (2 connections now open)
2016-09-14T19:11:32.894-0700 I ACCESS   [conn115]  authenticate db: admin { authenticate: 1, user: "admin", nonce: "xxx", key: "xxx" }
2016-09-14T19:11:32.894-0700 I ACCESS   [conn115] Failed to authenticate admin@admin with mechanism MONGODB-CR: AuthenticationFailed: MONGODB-CR credentials missing in the user document
2016-09-14T19:11:32.895-0700 I NETWORK  [conn115] end connection 192.168.94.130:48657 (1 connection now open)
2016-09-14T19:11:54.283-0700 I NETWORK  [initandlisten] connection accepted from 192.168.94.130:48663 #116 (2 connections now open)
2016-09-14T19:11:54.284-0700 I NETWORK  [conn116] end connection 192.168.94.130:48663 (1 connection now open)
2016-09-14T19:12:07.860-0700 I NETWORK  [initandlisten] connection accepted from 192.168.94.130:48666 #117 (2 connections now open)
2016-09-14T19:12:07.861-0700 I ACCESS   [conn117] Unauthorized: not authorized on admin to execute command { listDatabases: 1 }
```
Solution.

1. Delete exsisting users on the database if it was already created.
2. Modify the collection `admin.system.version` such that the `authSchema`  `currentVersion` is `3` instead of `5`
3. Version `3` is using MongoDB-CR
4. Recreate your user on the databases.

NOTE : Do not do it on PRODUCTION environment, use `update` instead and try on `test` database first.

``` ruby
mongo
use admin
db.system.users.remove({})
db.system.version.remove({})
db.system.version.insert({ "_id" : "authSchema", "currentVersion" : 3 })
```

More Details Here:

-  [`http://stackoverflow.com/a/31476552`](http://stackoverflow.com/a/31476552)
- [`http://stackoverflow.com/a/29193735`](http://stackoverflow.com/a/29193735)
