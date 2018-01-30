---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: Installing Zabbix Version 2.4 Offline (Zabbix Server without Internet).
category: ['Linux', 'Zabbix', 'Snmp', 'Trapper', 'Nagios', 'Monitoring']
tags: ['zabbix', 'snmp', 'trap', 'centos', 'linux', 'offline', 'nagios', 'monitoring']
---

There might be situations where you have a remote/zabbix server which does not have internet connectivity, due to security or other reasons. 
So we create a custom repo on the remote/zabbix server so that we can install zabbix using `rpms`

Here is how we are planning to do this.

1. Download all the dependency `rpms` on a machine which has internet connection, using `yum-downloadonly` or `repotrack`.
2. Transfer all the `rpms` to the remote server.
3. Create a `repo` on the remote server.
4. Update yum configuration.
5. Install.

**NOTE: This method can be used to install any application, but here we have used `zabbix` as we had this requirement for a zabbix server.**

###  Download dependent `rpms`.

On a machine which has internet connection install the package below. And download all the `rpms`.
Make sure the system are similar (not required to be identical - At-least the OS should be of same version)

	mkdir /zabbix_rpms
	yum install yum-downloadonly

Downloading all the rpms to location `/zabbix_rpms/`, `--downloadonly ` will only download the package but not install them. `--downloadonly` option to set downloadonly, `--downloaddir=/zabbix_rpms/` setting path to download.

	yum install mysql-server mysql -y --downloadonly --downloaddir=/zabbix_rpms/
	yum install zabbix-server-mysql zabbix-web-mysql -y --downloadonly --downloaddir=/zabbix_rpms/
	yum install zabbix-agent  -y --downloadonly --downloaddir=/zabbix_rpms/
	yum install createrepo -y --downloadonly --downloaddir=/zabbix_rpms/

To download all dependent `rpms` use `repotrack`.	
	
	repotrack -a x86_64 -p /zabbix_rpms/ [package]

There was a dependency on the remote server which was not resolving. So we download all the `rpms` recursively which resolved the issue.
	 
	[root@internet-access-server ahmed]# repotrack -a x86_64 -p /zabbix_rpms/  lm_sensors
	Downloading basesystem-10.0-4.el6.noarch.rpm
	Downloading bash-4.1.2-33.el6_7.1.x86_64.rpm
	Downloading chkconfig-1.3.49.3-5.el6.x86_64.rpm
	Downloading db4-4.7.25-20.el6_7.x86_64.rpm
	Downloading dmidecode-2.12-6.el6.x86_64.rpm
	Downloading filesystem-2.4.30-3.el6.x86_64.rpm
	Downloading gdbm-1.8.0-38.el6.x86_64.rpm
	Downloading glibc-2.12-1.166.el6_7.3.i686.rpm
	Downloading glibc-2.12-1.166.el6_7.3.x86_64.rpm
	Downloading glibc-common-2.12-1.166.el6_7.3.x86_64.rpm
	Downloading libattr-2.4.44-7.el6.x86_64.rpm
	Downloading libcap-2.16-5.5.el6.x86_64.rpm
	Downloading libgcc-4.4.7-16.el6.i686.rpm
	Downloading libgcc-4.4.7-16.el6.x86_64.rpm
	Downloading libselinux-2.0.94-5.8.el6.x86_64.rpm
	Downloading libsepol-2.0.41-4.el6.i686.rpm
	Downloading libsepol-2.0.41-4.el6.x86_64.rpm
	Downloading lm_sensors-3.1.1-17.el6.x86_64.rpm
	Downloading lm_sensors-libs-3.1.1-17.el6.x86_64.rpm
	Downloading ncurses-base-5.7-4.20090207.el6.x86_64.rpm
	Downloading ncurses-libs-5.7-4.20090207.el6.x86_64.rpm
	Downloading ncurses-libs-5.7-4.20090207.el6.i686.rpm
	Downloading nss-softokn-freebl-3.14.3-23.el6_7.i686.rpm
	Downloading nss-softokn-freebl-3.14.3-23.el6_7.x86_64.rpm
	Downloading perl-5.10.1-141.el6_7.1.x86_64.rpm
	Downloading perl-Module-Pluggable-3.90-141.el6_7.1.x86_64.rpm
	Downloading perl-Perlilog-0.3-4.el6.noarch.rpm
	Downloading perl-Pod-Escapes-1.04-141.el6_7.1.x86_64.rpm
	Downloading perl-Pod-Simple-3.13-141.el6_7.1.x86_64.rpm
	Downloading perl-libs-5.10.1-141.el6_7.1.x86_64.rpm
	Downloading perl-libs-5.10.1-141.el6_7.1.i686.rpm
	Downloading perl-version-0.77-141.el6_7.1.x86_64.rpm
	Downloading popt-1.13-7.el6.x86_64.rpm
	Downloading setup-2.8.14-20.el6_4.1.noarch.rpm
	Downloading tzdata-2015g-2.el6.noarch.rpm
	[root@internet-access-server ahmed]# cd /repos/packages/	 
	
###  Transfer all `rpms` to the remote server.

First we archive it.

	[root@internet-access-server /]# tar czf zabbix_rpms.tgz zabbix_rpms

Now send the archived file.

	[root@internet-access-server /]# scp zabbix_rpms.tgz root@10.222.73.88:/tmp/
	The authenticity of host '10.222.73.88 (10.222.73.88)' can't be established.
	RSA key fingerprint is ed:a9:e2:50:6d:45:5b:bb:0f:2e:53:90:ee:86:f7:26.
	Are you sure you want to continue connecting (yes/no)? yes
	Warning: Permanently added '10.222.73.88' (RSA) to the list of known hosts.
	root@10.222.73.88's password:
	zabbix_rpms.tgz                                                     100%   23MB  30.2KB/s   13:05
	[root@internet-access-server /]#


###  Create a `repo` on the remote server.

	rpm -ivh deltarpm-3.5-0.5.20090913git.el6.x86_64.rpm
	rpm -ivh python-deltarpm-3.5-0.5.20090913git.el6.x86_64.rpm
	rpm -ivh createrepo-0.9.9-22.el6.noarch.rpm

Executing output.	
	
	[root@remote-zabbix-server ZBX_RPMS]# rpm -ivh deltarpm-3.5-0.5.20090913git.el6.x86_64.rpm
	warning: deltarpm-3.5-**6.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID c105b9de: NOKEY
	Preparing...                ########################################### [100%]
	   1:deltarpm               ########################################### [100%]
	[root@remote-zabbix-server ZBX_RPMS]# rpm -ivh python-deltarpm-3.5-0.5.20090913git.el6.x86_64.rpm
	warning: python-delta*t.el6.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID c105b9de: NOKEY
	Preparing...                ########################################### [100%]
	   1:python-deltarpm        ########################################### [100%]
	[root@remote-zabbix-server ZBX_RPMS]# rpm -ivh createrepo-0.9.9-22.el6.noarch.rpm
	warning: createrepo-**ch.rpm: Header V3 RSA/SHA1 Signature, key ID c105b9de: NOKEY
	Preparing...                ########################################### [100%]
	   1:createrepo             ########################################### [100%]

Create directory for RPMS.

	mkdir -p /custom_repo/yum-channels/custom-repo-channel-sys/x86_64/

Create repo now.

	createrepo /custom_repo/yum-channels/custom-repo-channel-sys/x86_64/
	
###  Update yum configuration.

Update configuration to make the new `repo` available to `yum`.

####  Setting up repo in `/etc/yum.repos.d` Location.

Create a file call `custom-channel.repo` here.
	   
	[custom-repo-channel-appsrc-repo] 
	name=Custom Channel Sys Source Repository [ZABBIX]
	baseurl=file:///custom_repo/yum-channels/custom-repo-channel-sys/x86_64
	gpgcheck=0
	enabled=1
	proxy=_none_ 

####  Checking for the new repo added.

First we clean the repo.

	[root@remote-zabbix-server x86_64]# yum clean all
	Loaded plugins: product-id, security, subscription-manager
	Cleaning repos: custom-repo-channel-appsrc-repo
	Cleaning up Everything
	
Updating `repolist` now.
	
	[root@remote-zabbix-server x86_64]# yum repolist
	Loaded plugins: product-id, security, subscription-manager
	custom-repo-channel-appsrc-repo                                               | 2.9 kB     00:00 ...
	custom-repo-channel-appsrc-repo/primary_db                                    |  27 kB     00:00 ...
	repo id                              repo name                                           status
	custom-repo-channel-appsrc-repo      Custom Channel Sys Source Repository [ZABBIX]        28
	repolist: 28

Checking for `rpms` from the newly created `repo`.
	
	[root@remote-zabbix-server x86_64]# yum list zabbix-server-mysql
	Loaded plugins: product-id, security, subscription-manager
	Available Packages
	zabbix-server-mysql.x86_64          2.4.7-1.el6                   custom-repo-channel-appsrc-repo
	[root@remote-zabbix-server x86_64]#


###  Now we are ready to "Install".

Standard installation instrutions in the [link below](http://zubayr.github.io/zabbix-install-centos/).

	http://zubayr.github.io/zabbix-install-centos/