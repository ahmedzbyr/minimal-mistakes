---
title: Installing CouchDB on Ubuntu 14 LTS.
category: ['Linux', 'Couchdb', 'Ubuntu']
tags: ['linux', 'couchdb', 'ubuntu', 'nagios']
---
CouchDB is a database that completely embraces the web. Store your data with JSON documents. Access your documents and query your indexes with your web browser, via HTTP. Index, combine, and transform your documents with JavaScript. CouchDB works well with modern web and mobile apps. You can even serve web apps directly out of CouchDB. And you can distribute your data, or your apps, efficiently using CouchDB’s incremental replication. CouchDB supports master-master setups with automatic conflict detection.


## Installing CouchDB.

### Setting up Repos and Packages.

``` ruby
sudo apt-get install software-properties-common -y
sudo add-apt-repository ppa:couchdb/stable -y
sudo apt-get update -y
```

### Remove any exsisting installations.

``` ruby
sudo apt-get remove couchdb couchdb-bin couchdb-common -yf
```

### Installation.

``` ruby
sudo apt-get install -V couchdb
  Reading package lists...
  Done Building dependency tree
  Reading state information...
  Done
  The following extra packages will be installed:
  ...
  Y
```

### Stop and configure `couchdb`

``` ruby
sudo stop couchdb
  couchdb stop/waiting
```

#### update `/etc/couchdb/local.ini` with `bind_address=0.0.0.0` as needed

``` ruby
sudo start couchdb
  couchdb start/running, process 3541
```

#### Start Server
``` ruby
sudo stop couchdb
  couchdb stop/waiting
```
Finally we can go to the browser and check the server is up.
Apache CouchDB has started on `http://couchdb-server:5984/`
