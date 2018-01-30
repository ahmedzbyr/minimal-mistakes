---
title: YUM Repository Creation on HTTPD Web Server.
category: ['Linux', 'Hadoop', 'Httpd']
tags: ['linux', 'hadoop', 'webserver', 'http', 'httpd', 'yum', 'rhel', 'centos']
---

Setting up `yum` repos on RHEL using httpd. We will be setting up `httpd` and `yum repo` on top of it.
So that we can access `yum` over `http`. 

## `yum` Repository Creation on using `httpd` Web Server.

###  Installing prerequisites.

To setup a repository we need to install `createrepo` package.

    yum install createrepo.

###  Create directories to store the repo.

Below are the directories to be created, to store the rpms in.

    mkdir -p /export/custom-repo/yum-channels/custom-repo-channel-app/SRPMS
    mkdir -p /export/custom-repo/yum-channels/custom-repo-channel-app/x86_64
    mkdir -p /export/custom-repo/yum-channels/custom-repo-channel-sys/SRPMS
    mkdir -p /export/custom-repo/yum-channels/custom-repo-channel-sys/x86_64
    
###  Create subdirectories as below.
    
    [root@repo-server ~]# ls -l /export/custom-repo/
    total 12
    drwxrwxr-x 2 repomgr repomgr 4096 Oct 14 09:06 pkg
    drwxr-xr-x 2 root    root    4096 Oct 13 16:18 repodata
    drwxr-xr-x 5 repomgr repomgr 4096 Oct 14 11:09 yum-channels
    [root@repo-server ~]# ls -l /export/custom-repo/yum-channels/
    total 20
    drwxr-xr-x 4 repomgr repomgr 4096 Oct 14 11:09 custom-repo-channel-app
    drwxr-xr-x 4 repomgr repomgr 4096 Oct 14 10:53 custom-repo-channel-sys
    -rw-r--r-- 1 repomgr repomgr  150 Oct 14 11:00 Makefile
    -rw-r--r-- 1 repomgr repomgr   50 Oct 14 11:01 README
    [root@repo-server ~]# ls -l /export/custom-repo/yum-channels/custom-repo-channel-app/
    total 12
    -rw-r--r-- 1 repomgr repomgr  157 Oct 14 10:53 Makefile
    drwxr-xr-x 3 repomgr repomgr 4096 Oct 14 11:09 SRPMS
    drwxr-xr-x 3 repomgr repomgr 4096 Oct 14 11:00 x86_64
    [root@repo-server ~]# ls -l /export/custom-repo/yum-channels/custom-repo-channel-sys/
    total 12
    -rw-r--r-- 1 repomgr repomgr  157 Oct 14 10:53 Makefile
    drwxr-xr-x 3 repomgr repomgr 4096 Oct 14 11:00 SRPMS
    drwxr-xr-x 3 repomgr repomgr 4096 Oct 14 11:00 x86_64
    [root@repo-server ~]#
    
###  Creating Repository.
    
    createrepo /export/custom-repo/yum-channels/custom-repo-channel-app/SRPMS
    createrepo /export/custom-repo/yum-channels/custom-repo-channel-app/x86_64
    createrepo /export/custom-repo/yum-channels/custom-repo-channel-sys/SRPMS
    createrepo /export/custom-repo/yum-channels/custom-repo-channel-sys/x86_64

Here is how the directory looks after createrepo

    [root@repo-server ~]# tree /export/custom-repo/yum-channels/custom-repo-channel-sys/
    /export/custom-repo/yum-channels/custom-repo-channel-sys/
    ├── Makefile
    ├── SRPMS
    │   └── repodata
    │       ├── 401dc19bda88c82c40347e95f5b9835888189c03834cc93-filelists.xml.gz
    │       ├── 6bf9672d0862e8ef8b8ff5978e6589d87944c88259cb670-other.xml.gz
    │       ├── 77a287c136f4ff47df50a525f06ddf41a3fef39908d61a7-other.sqlite.bz2
    │       ├── 8596812757300b1d87f2d8ee28c11009e5980cb5cd4be14-primary.sqlite.bz2
    │       ├── dabe2ce5481d23de1f4f6c9e0de2ce8123adeefa0dd08b9-primary.xml.gz
    │       ├── f8606d9f21d61a8bf40502becb78ba5fea7d0f1cd06a0b2-filelists.sqlite.bz2
    │       └── repomd.xml
    └── x86_64
        └── repodata
            ├── 401dc19bda88c82c40347e95f5b9835888189c03834cc93-filelists.xml.gz
            ├── 6bf9672d0862e8ef8b8ff5978e6589d87944c88259cb670-other.xml.gz
            ├── 77a287c136f4ff47df50a525f06ddf41a3fef39908d61a7-other.sqlite.bz2
            ├── 8596812757300b1d87f2d8ee28c11009e5980cb5cd4be14-primary.sqlite.bz2
            ├── dabe2ce5481d23de1f4f6c9e0de2ce8123adeefa0dd08b9-primary.xml.gz
            ├── f8606d9f21d61a8bf40502becb78ba5fea7d0f1cd06a0b2-filelists.sqlite.bz2
            └── repomd.xml

    4 directories, 15 files

##  Installing and configuring httpd to host the yum repository.

Setting up httpd web server, and configure it to host the repos.

###  Install httpd

    yum install httpd

###  Configuration.

Create a file in `/etc/httpd/conf.d/` as `custom-repo-channel-app.conf`.

    Alias /customrepochannelapp "/export/custom-repo/yum-channels/custom-repo-channel-app"
    <Directory "/export/custom-repo/yum-channels/custom-repo-channel-app">
        Options +Indexes
        Allow from all
    </Directory>

`httpd` configuration for the server for all the repos.

    [root@repo-server ~]# cat /etc/httpd/conf.d/
    custom-repo-channel-app.conf  custom-repo-channel-sys.conf  README              welcome.conf
    [root@repo-server ~]# cat /etc/httpd/conf.d/custom-repo-channel-*
    Alias /customrepochannelapp "/export/custom-repo/yum-channels/custom-repo-channel-app"
    <Directory "/export/custom-repo/yum-channels/custom-repo-channel-app">
        Options +Indexes
        Allow from all
    </Directory>

    Alias /customrepochannelsys "/export/custom-repo/yum-channels/custom-repo-channel-sys"
    <Directory "/export/custom-repo/yum-channels/custom-repo-channel-sys">
        Options +Indexes
        Allow from all
    </Directory>

    [root@repo-server ~]#

Restart httpd server.

    service httpd restart

###  Configuring yum.

Create a file `custom-channel.repo` in `/etc/yum.repos.d/`.

For  we are using the below to configure the server. If this repo is inside the network which does not need a proxy then, make sure to use `proxy=_none_` as  we do NOT want the repo to use it.

    [custom-repo-channel-appsrc-repo]
    name=Custom Channel App Source Repository
    baseurl=http://repo-server.myorg.com/customrepochannelapp/SRPMS
    gpgcheck=0
    enabled=1
    proxy=_none_

Complete list of rpms configured as below.

    [root@repo-server ~]# cat /etc/yum.repos.d/custom-channel.repo
    [custom-repo-channel-appsrc-repo]
    name=Custom Channel App Source Repository
    baseurl=http://repo-server.myorg.com/customrepochannelapp/SRPMS
    gpgcheck=0
    enabled=1
    proxy=_none_

    [custom-repo-channel-app-repo]
    name=Custom Channel App Repository
    baseurl=http://repo-server.myorg.com/customrepochannelapp/x86_64
    gpgcheck=0
    enabled=1
    proxy=_none_

    [custom-repo-channel-syssrc-repo]
    name=Custom Channel System Source Repository
    baseurl=http://repo-server.myorg.com/customrepochannelsys/SRPMS
    gpgcheck=0
    enabled=1
    proxy=_none_

    [custom-repo-channel-sys-repo]
    name=Custom Channel System Repository
    baseurl=http://repo-server.myorg.com/customrepochannelsys/x86_64
    gpgcheck=0
    enabled=1
    proxy=_none_
    [root@repo-server ~]#

Here is the output for repolist.

    [root@repo-server ~]# yum repolist
    Loaded plugins: product-id, rhnplugin, security, subscription-manager
    repo id                                             repo name                       status
    custom-repo-channel-app-repo         Custom Channel App Repository                    112
    custom-repo-channel-appsrc-repo      Custom Channel App Source Repository              12
    custom-repo-channel-sys-repo         Custom Channel System Repository                 144
    custom-repo-channel-syssrc-repo      Custom Channel System Source Repository           44
    rhel-6-server-rpms                   Red Hat Enterprise Linux 6 Server (RPMs)      16,167
    rhel-server-dts-6-rpms               Red Hat Developer Toolset RPMs for Red Ha         84
    rhel-server-dts2-6-rpms              Red Hat Developer Toolset 2 RPMs for Red         469
    repolist: xx
    [root@repo-server ~]#

