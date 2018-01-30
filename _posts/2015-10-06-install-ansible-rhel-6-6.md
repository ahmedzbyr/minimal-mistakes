---
title: Installing `ansible` on RHEL 6.6.
category: ['Hadoop', 'Linux', 'Ansible']
tags: ['linux', 'ansible', 'hadoop', 'rhel', 'centos']
---

Ansible is a radically simple IT automation engine that automates cloud provisioning, configuration management, application deployment, intra-service orchestration, and many other IT needs. For more detail, hop over to `docs.ansible.com`.

Download `epel` and install.

	[root@server-cloudera-manager ~]# wget \
                    http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
	[root@server-cloudera-manager ~]# rpm -ivh epel-release-6-8.noarch.rpm
	warning: epel-release-6-8.noarch.rpm: Header V3 RSA/SHA256 Signature, key ID 0608b895: NOKEY
	Preparing...                ########################################### [100%]
	   1:epel-release           /etc/yum.repos.d/epel.repo
	########################################### [100%]

**IMPORTANT** : As of now in redhat 6.6 `python-jinja2` is moved to `optional` repo in redhat from `epel`. So we need to enable `optional` repo in redhat.

* Information on `python-jinja2` move from EPEL: `http://docs.saltstack.com/en/latest/topics/installation/rhel.html` 

* How to enable repo: `https://access.redhat.com/solutions/265523`

ERROR before we enable the optional rpms.

    [root@server ~]# yum install ansible
    Loaded plugins: product-id, rhnplugin, security, subscription-manager
    epel/metalink                                                                 |  12 kB     00:00
    epel                                                                          | 4.3 kB     00:00
    epel/primary_db                                                               | 5.7 MB     00:43
    rhel-6-server-rpms                                                            | 3.7 kB     00:00
    rhel-6-server-rpms/primary_db                                                 |  35 MB     01:29
    rhel-server-dts-6-rpms                                                        | 2.9 kB     00:00
    rhel-server-dts2-6-rpms                                                       | 2.9 kB     00:00
    Resolving Dependencies
    --> Running transaction check
    ---> Package ansible.noarch 0:1.9.2-1.el6 will be installed
    --> Processing Dependency: python-simplejson for package: ansible-1.9.2-1.el6.noarch
    --> Processing Dependency: python-keyczar for package: ansible-1.9.2-1.el6.noarch
    --> Processing Dependency: python-jinja2 for package: ansible-1.9.2-1.el6.noarch
    --> Processing Dependency: python-httplib2 for package: ansible-1.9.2-1.el6.noarch
    --> Processing Dependency: python-crypto2.6 for package: ansible-1.9.2-1.el6.noarch
    --> Processing Dependency: PyYAML for package: ansible-1.9.2-1.el6.noarch
    --> Running transaction check
    ---> Package PyYAML.x86_64 0:3.10-3.1.el6 will be installed
    --> Processing Dependency: libyaml-0.so.2()(64bit) for package: PyYAML-3.10-3.1.el6.x86_64
    ---> Package ansible.noarch 0:1.9.2-1.el6 will be installed
    --> Processing Dependency: python-jinja2 for package: ansible-1.9.2-1.el6.noarch
    ---> Package python-crypto2.6.x86_64 0:2.6.1-2.el6 will be installed
    ---> Package python-httplib2.noarch 0:0.7.7-1.el6 will be installed
    ---> Package python-keyczar.noarch 0:0.71c-1.el6 will be installed
    --> Processing Dependency: python-pyasn1 for package: python-keyczar-0.71c-1.el6.noarch
    ---> Package python-simplejson.x86_64 0:2.0.9-3.1.el6 will be installed
    --> Running transaction check
    ---> Package ansible.noarch 0:1.9.2-1.el6 will be installed
    --> Processing Dependency: python-jinja2 for package: ansible-1.9.2-1.el6.noarch
    ---> Package libyaml.x86_64 0:0.1.3-4.el6_6 will be installed
    ---> Package python-pyasn1.noarch 0:0.0.12a-1.el6 will be installed
    --> Finished Dependency Resolution
    Error: Package: ansible-1.9.2-1.el6.noarch (epel)
               Requires: python-jinja2
     You could try using --skip-broken to work around the problem
     You could try running: rpm -Va --nofiles --nodigest
    [root@server ~]#



Here is more verbose from the installation. 
 
Enable optional repo.

	[root@server-cloudera-manager yum.repos.d]# subscription-manager repos \
                                                                --enable=rhel-6-server-optional-rpms

Install `ansible`.

	[root@server-cloudera-manager ~]# yum install ansible
	
	===========================================================================
	 Package           Arch   Version        Repository                   Size
	===========================================================================
	Installing:
	 ansible           noarch 1.9.2-1.el6    epel                        1.7 M
	Installing for dependencies:
	 PyYAML            x86_64 3.10-3.1.el6   rhel-6-server-rpms          157 k
	 libyaml           x86_64 0.1.3-4.el6_6  rhel-6-server-rpms           52 k
	 python-babel      noarch 0.9.4-5.1.el6  rhel-6-server-rpms          1.4 M
	 python-crypto2.6  x86_64 2.6.1-2.el6    epel                        513 k
	 python-httplib2   noarch 0.7.7-1.el6    epel                         70 k
	 python-jinja2     x86_64 2.2.1-2.el6_5  rhel-6-server-optional-rpms 466 k
	 python-keyczar    noarch 0.71c-1.el6    epel                        219 k
	 python-pyasn1     noarch 0.0.12a-1.el6  rhel-6-server-rpms           70 k
	 python-simplejson x86_64 2.0.9-3.1.el6  rhel-6-server-rpms          126 k

	Transaction Summary
	===========================================================================
	Install      10 Package(s)

	
	[root@server-cloudera-manager ~]# ansible --version
	ansible 1.9.2
	  configured module search path = None
	[root@server-cloudera-manager ~]#
