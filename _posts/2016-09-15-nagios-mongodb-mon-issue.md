---
title: Issues - Monitoring MongoDB using Nagios XI.
category: ['Linux', 'Mongodb', 'Ubuntu', 'Nagios', 'Nagiosxi']
tags: ['linux', 'mongo', 'mongodb', 'ubuntu', 'nagios', 'nagiosxi']
---

Monitoring for `mongodb` using `nagiosxi` is straight forword but you might have some issues when we are setting up.
Here are few issues which might come up using `mongodb` version `3`.


### Issue getting monitoring data in `nagios`.

#### 1. `ConnectionFailure` object has no attribute `strip`

``` ruby
[ahmed@localhost libexec]$ ./check_mongodb.py -H 192.168.94.137 -P 27017 -u admin -p admin
Traceback (most recent call last):
  File "./check_mongodb.py", line 1372, in <module>
    sys.exit(main(sys.argv[1:]))
  File "./check_mongodb.py", line 196, in main
    err, con = mongo_connect(host, port, ssl, user, passwd, replicaset)
  File "./check_mongodb.py", line 294, in mongo_connect
    return exit_with_general_critical(e), None
  File "./check_mongodb.py", line 310, in exit_with_general_critical
    if e.strip() == "not master":
AttributeError: 'ConnectionFailure' object has no attribute 'strip'
```

Solution.

`e.strip()` expects `e` to be a string, which might not be the case sometimes, so remove `strip()`. Change below code on line `310`.

``` python
  else:
      if e.strip() == "not master":
          print "UNKNOWN - Could not get data from server:", e
          return 3
```

to

``` python
  else:
      if e == "not master":
          print "UNKNOWN - Could not get data from server:", e
          return 3
```

After the change atleast you will get an `error` which gives you more information.

``` ruby
[ahmed@localhost libexec]$ ./check_mongodb_2.py -H 192.168.94.138 -P 27017 -u admin -p admin1 -A databases -W 5 -C 10
CRITICAL - General MongoDB Error: command SON([('authenticate', 1), ('user', u'admin'), ('nonce', u'37a502d665186449'), ('key', u'd8c683f98a5e720c28a8007018ed7414')]) failed: auth failed
```

Next we will try to resolve, above auth failure.


#### 2. Executing command from the `nagios` server.

``` ruby
[ahmed@localhost libexec]$ ./check_mongodb_2.py -H 192.168.94.138 -P 27017 -u admin -p admin1 -A databases -W 5 -C 10
CRITICAL - General MongoDB Error: command SON([('authenticate', 1), ('user', u'admin'), ('nonce', u'42110dc29ee7fe6b'), ('key', u'827a2b0e4af97e88560800ab86b04e57')]) failed: auth failed
```


#### On the mongodb server.

Checking on the mongodb server shows that the `AuthenticationFailed` due to `MONGODB-CR credentials missing in the user document`

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
