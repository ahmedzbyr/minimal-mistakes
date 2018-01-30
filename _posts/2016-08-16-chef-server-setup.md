---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: Standalone Chef Server / Workstation / Client Setup on CentOS 6
category: ['Linux', 'Centos', 'Chef', 'Chefdk']
tags: ['centos', 'linux', 'chef', 'chefdk']
---

The standalone installation of Chef server creates a working installation on a single server. This installation is also useful when you are installing Chef server in a virtual machine, for proof-of-concept deployments, or as a part of a development or testing loop.

## Before we start.

[Install the Chef Server - Prerequisites](https://docs.chef.io/release/server_12-6/install_server.html)

In our environment we will be creating the below nodes.

    192.168.30.132  chefserver.ahmed.com    
    192.168.30.142  chefworkstation.ahmed.com    
    192.168.30.141  chefnode.ahmed.com 


Setup the `/etc/hosts` file on all the nodes.

    127.0.0.1   localhost
    192.168.30.132  chefserver.ahmed.com    chefserver
    192.168.30.142  chefworkstation.ahmed.com    chefworkstation
    192.168.30.141  chefnode.ahmed.com    chefnode

Install `crontabs` (if not already installed)

	yum install crontabs
    
Flush all the `iptables` rules and save them on all Nodes. 

    iptables -F
    service iptables save

Command output.

    [root@chefnode Downloads]# iptables -F
    [root@chefnode Downloads]# service iptables save
    iptables: Saving firewall rules to /etc/sysconfig/iptables:[  OK  ]
    [root@chefnode Downloads]# iptables -L
    Chain INPUT (policy ACCEPT)
    target     prot opt source               destination

    Chain FORWARD (policy ACCEPT)
    target     prot opt source               destination

    Chain OUTPUT (policy ACCEPT)
    target     prot opt source               destination
    [root@chefnode Downloads]#
    
Set `selinux` to `Permissive` mode on all nodes.
    
    setenforce Permissive
    getenforce

Command output.

    [root@chefnode Downloads]# setenforce Permissive
    [root@chefnode Downloads]# getenforce
    Permissive
    [root@chefnode Downloads]#


Download the package from [http://downloads.chef.io/chef-server/](http://downloads.chef.io/chef-server/).

    [root@chefnode Downloads]# ls -l
    total 656944
    -rwxr--r--. 1 root root  52338858 Jun  8 01:45 chef-12.10.24-1.el6.x86_64.rpm
    -rwxr--r--. 1 root root 142488123 Jun  8 01:45 chefdk-0.14.25-1.el6.x86_64.rpm
    -rwxr--r--. 1 root root 477870942 Jun  8 01:45 chef-server-core-12.6.0-1.el6.x86_64.rpm
    [root@chefnode Downloads]#

## Creating `chef-server`.

Install the package as below.    

    rpm -ivh chef-server-core-12.6.0-1.el6.x86_64.rpm
    
First thing we need to do is to reconfigure the server.

    chef-server-ctl reconfigure

---

**NOTE : If you get the below error.**

**Problem**

	[2015-09-23T16:46:46+00:00] ERROR: Exception handlers complete
	  Chef Client failed. 1 resources updated in 10.379691033 seconds
	[2015-09-23T16:46:46+00:00] FATAL: Stacktrace dumped to /opt/opscode/embedded/cookbooks/cache/chef-stacktrace.out
	[2015-09-23T16:46:47+00:00] FATAL: Chef::Exceptions::ValidationFailed: common_name is required

**Solution** 

* Make sure you have the `hostname` in the `/etc/hosts`, hostname should resolve to an IP address does not matter if it is loopback as well.  
* Change the `hostname`, `sudo hostname chefserver`, here `chefserver` is the new hostname.
* Also change the `hostname` in the file `/etc/sysconfig/network` so that the name persists after a restart. Change `HOSTNAME=chefserver` in the file. 
* Added the host name `sudo echo "127.0.0.1 localhost chefserver" >> /etc/hosts`.


**Problem** 
	
	[2016-02-10T22:09:53+00:00] FATAL:
	
	-----------------------------------------------------------------------
	BOOT007: The secrets file (/etc/opscode/private-chef-secrets.json) is present
	         but the file /etc/opscode/pivotal.pem is missing.
	
	         Ensure that private-chef-secrets.json is copied into /etc/opscode from the
	         first Chef Server node that you brought online, then run
	         'chef-server-ctl reconfigure' again.
	-----------------------------------------------------------------------

**Solution**
    
* Rename `/etc/opscode/private-chef-secrets.json` to `/etc/opscode/private-chef-secrets.json.org` and try if the problem still persists then try the next option.

**`---OR---(if the above solution does not work)---`**

* For `/etc/opscode/pivotal.pem` is missing error.

Use below command.

	cp /opt/opscode/embedded/service/omnibus-ctl/spec/fixtures/pivotal.pem /etc/opscode/

---

Create a user for the server.
    
    chef-server-ctl user-create USER_NAME FIRST_NAME LAST_NAME EMAIL 'PASSWORD' --filename PATH_TO_FILE_NAME

Command.
    
    chef-server-ctl user-create ahmed Zubair AHMED zubayr.a@gmail.com 'ahmed@123' --filename /etc/chef/ahmed.pem 

Create an organization.

    chef-server-ctl org-create short_name 'full_organization_name' --association_user user_name --filename ORGANIZATION-validator.pem 

Command.
    
    chef-server-ctl org-create ahmedinc 'Ahmed, Inc.' --association_user ahmed --filename /etc/chef/ahmed-validator.pem

Setting up and installing chef-manage.

    chef-server-ctl install chef-manage
    chef-server-ctl reconfigure
    chef-manage-ctl reconfigure

Check server status using below command.

    
    chef-server-ctl status

we see all the services running fine.

    [root@chefserver Downloads]# chef-server-ctl status
    run: bookshelf: (pid 6081) 4002s; run: log: (pid 6121) 4001s
    run: nginx: (pid 8725) 2724s; run: log: (pid 6245) 3995s
    run: oc_bifrost: (pid 5924) 4008s; run: log: (pid 5959) 4008s
    run: oc_id: (pid 5976) 4006s; run: log: (pid 5984) 4006s
    run: opscode-erchef: (pid 6192) 3999s; run: log: (pid 6160) 4001s
    run: opscode-expander: (pid 6046) 4002s; run: log: (pid 6064) 4002s
    run: opscode-solr4: (pid 6007) 4004s; run: log: (pid 6036) 4003s
    run: postgresql: (pid 5901) 4009s; run: log: (pid 5914) 4009s
    run: rabbitmq: (pid 5814) 4010s; run: log: (pid 5807) 4010s
    run: redis_lb: (pid 8133) 2805s; run: log: (pid 6241) 3996s
    [root@chefserver Downloads]#
 
Now we can go the the browser and check our installation.

NOTE : if you see an certificate error, then added the server to the exceptions, this is as we are using `https` and we dont have a valid certificate.

	https://chefserver.ahmed.com/

**Login on to the server.** 

![login to the server](http://zubayr.github.io/images/chef_server_0.1_install_complete.PNG)

**Nodes on the server - First Screen.**

![Nodes on the server](http://zubayr.github.io/images/chef_server_0.2_install_complete.PNG)

**Current configured users.**

![Users on the server](http://zubayr.github.io/images/chef_server_0.3_install_complete.PNG)
   
Next we will setup the `workstation` node.

## Installing `chefdk` on workstation node.  

More information [here](https://docs.chef.io/release/12-5/install_dk.html)

### Install `chefdk-0.14.25-1.el6.x86_64.rpm` on the node. 
  
    [root@chefworkstation Downloads]# rpm -ivh chefdk-0.14.25-1.el6.x86_64.rpm
    warning: chefdk-0.14.25-1.el6.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID
    Preparing...                ########################################### [100%]
       1:chefdk                 ########################################### [100% ]
    Thank you for installing Chef Development Kit!

    
Check for selinux.

    [root@chefworkstation Downloads]# cat /etc/selinux/config

    # This file controls the state of SELinux on the system.
    # SELINUX= can take one of these three values:
    #     enforcing - SELinux security policy is enforced.
    #     permissive - SELinux prints warnings instead of enforcing.
    #     disabled - No SELinux policy is loaded.
    SELINUX=enforcing
    # SELINUXTYPE= can take one of these two values:
    #     targeted - Targeted processes are protected,
    #     mls - Multi Level Security protection.
    SELINUXTYPE=targeted

Set `selinux` to `Permissive` .   

    [root@chefworkstation Downloads]# setenforce Permissive
    [root@chefworkstation Downloads]# getenforce
    Permissive

Check installation using `chef verify`.

	[root@chefworkstation chefdk]# chef verify
	Running verification for component 'berkshelf'
	Running verification for component 'test-kitchen'
	Running verification for component 'tk-policyfile-provisioner'
	Running verification for component 'chef-client'
	Running verification for component 'chef-dk'
	Running verification for component 'chef-provisioning'
	Running verification for component 'chefspec'
	Running verification for component 'generated-cookbooks-pass-chefspec'
	Running verification for component 'rubocop'
	Running verification for component 'fauxhai'
	Running verification for component 'knife-spork'
	Running verification for component 'kitchen-vagrant'
	Running verification for component 'package installation'
	Running verification for component 'openssl'
	Running verification for component 'inspec'
	Running verification for component 'delivery-cli'
	Running verification for component 'git'
	Running verification for component 'opscode-pushy-client'
	Running verification for component 'chef-sugar'
	Running verification for component 'knife-supermarket'
	..............................................
	---------------------------------------------
	Verification of component 'fauxhai' succeeded.
	Verification of component 'kitchen-vagrant' succeeded.
	Verification of component 'openssl' succeeded.
	Verification of component 'delivery-cli' succeeded.
	Verification of component 'test-kitchen' succeeded.
	Verification of component 'rubocop' succeeded.
	Verification of component 'inspec' succeeded.
	Verification of component 'opscode-pushy-client' succeeded.
	Verification of component 'knife-supermarket' succeeded.
	Verification of component 'berkshelf' succeeded.
	Verification of component 'knife-spork' succeeded.
	Verification of component 'git' succeeded.
	Verification of component 'tk-policyfile-provisioner' succeeded.
	Verification of component 'chefspec' succeeded.
	Verification of component 'chef-sugar' succeeded.
	Verification of component 'chef-client' succeeded.
	Verification of component 'chef-dk' succeeded.
	Verification of component 'package installation' succeeded.
	Verification of component 'chef-provisioning' succeeded.
	Verification of component 'generated-cookbooks-pass-chefspec' succeeded.


Setting up `ruby`.
  
    [root@chefworkstation ahmed]# eval "$(chef shell-init bash)"
    [root@chefworkstation ahmed]# which ruby
    /opt/chefdk/embedded/bin/ruby
    [root@chefworkstation ahmed]#

To make these changes persist, execute below command to update `.bash_profile`.

	echo 'eval "$(chef shell-init bash)"' >> ~/.bash_profile


### Getting the starter kit.

Goto the `chef-server` > `Administration` Tab > Select `organization` > Click `Starter Kit` on the left pane.

**Admin Tab**

![files](http://zubayr.github.io/images/chef_server_organization_view.png)

**Select starter-kit**

![files](http://zubayr.github.io/images/chef_server_download_starter_kit.png)

**Reset keys and download**

![files](http://zubayr.github.io/images/chef_server_resetting_to_create_starter_kit.png)

Copy the starter kit (`chef-repo`) from the server and unzip and save it in the home directory on **WORKSTATION NODE**.
 
    [root@chefworkstation ahmed]# pwd
    /home/ahmed
    [root@chefworkstation ahmed]# ls -l
    total 40
    drwxr-xr-x. 5 root  root  4096 Jun  8 03:27 chef-repo
    drwxr-xr-x. 2 ahmed ahmed 4096 May  7  2015 Desktop
    drwxr-xr-x. 2 ahmed ahmed 4096 May  7  2015 Documents
    drwxr-xr-x. 2 ahmed ahmed 4096 Jun  8 02:55 Downloads
    drwxrwxr-x. 6 ahmed ahmed 4096 Feb  3 01:39 github
    drwxr-xr-x. 2 ahmed ahmed 4096 May  7  2015 Music
    drwxr-xr-x. 2 ahmed ahmed 4096 May  7  2015 Pictures
    drwxr-xr-x. 2 ahmed ahmed 4096 May  7  2015 Public
    drwxr-xr-x. 2 ahmed ahmed 4096 May  7  2015 Templates
    drwxr-xr-x. 2 ahmed ahmed 4096 May  7  2015 Videos
    [root@chefworkstation ahmed]# tree chef-repo/
    chef-repo/
    ├── cookbooks
    │   ├── chefignore
    │   ├── cm_setup
    │   │   ├── Berksfile
    │   │   ├── Berksfile.lock
    │   │   ├── chefignore
    │   │   ├── metadata.rb
    │   │   ├── README.md
    │   │   ├── recipes
    │   │   │   └── default.rb
    │   │   ├── spec
    │   │   │   ├── spec_helper.rb
    │   │   │   └── unit
    │   │   │       └── recipes
    │   │   │           └── default_spec.rb
    │   │   ├── templates
    │   │   │   └── default
    │   │   │       ├── cloudera_manager.repo.erb
    │   │   │       ├── my.cnf.erb
    │   │   │       └── sysctl.conf.erb
    │   │   └── test
    │   │       └── integration
    │   │           ├── default
    │   │           │   └── serverspec
    │   │           │       └── default_spec.rb
    │   │           └── helpers
    │   │               └── serverspec
    │   │                   └── spec_helper.rb
    │   └── starter
    │       ├── attributes
    │       │   └── default.rb
    │       ├── files
    │       │   └── default
    │       │       └── sample.txt
    │       ├── metadata.rb
    │       ├── recipes
    │       │   └── default.rb
    │       └── templates
    │           └── default
    │               └── sample.erb
    ├── README.md
    └── roles
        └── starter.rb

    22 directories, 21 files
    [root@chefworkstation ahmed]#

    
   
### Fetching the SSL key from the `chef-server`.

    [root@chefworkstation chef-repo]# knife ssl fetch
    WARNING: Certificates from chefserver.ahmed.com will be fetched and placed in your trusted_cert
    directory (/home/ahmed/chef-repo/.chef/trusted_certs).

    Knife has no means to verify these are the correct certificates. You should
    verify the authenticity of these certificates after downloading.

    Adding certificate for chefserver.ahmed.com in /home/ahmed/chef-repo/.chef/trusted_certs/chefserver_ahmed_com.crt

Check client currently available. Currently we see only the work station.

    [root@chefworkstation chef-repo]# knife client list
    ahmedinc-validator


## Bootstrapping a Node.

More information [here](https://docs.chef.io/release/12-5/install_bootstrap.html).

### Setting up a client using the `knife bootstrap` command.

    [root@chefworkstation chef-repo]#  knife bootstrap chefnode.ahmed.com -x ahmed -P ahmed --sudo
    Doing old-style registration with the validation key at /home/ahmed/chef-repo/.chef/ahmedinc-validator.pem...
    Delete your validation key in order to use your user credentials instead

    Connecting to chefnode.ahmed.com
    chefnode.ahmed.com This is BASH 4.1     - DISPLAY on
    chefnode.ahmed.com
    chefnode.ahmed.com Wed Jun  8 03:05:01 PDT 2016
    chefnode.ahmed.com -----> Existing Chef installation detected
    chefnode.ahmed.com Starting the first Chef Client run...
    chefnode.ahmed.com Starting Chef Client, version 12.10.24
    chefnode.ahmed.com Creating a new client identity for chefnode.ahmed.com using the validator key.
    chefnode.ahmed.com resolving cookbooks for run list: []
    chefnode.ahmed.com Synchronizing Cookbooks:
    chefnode.ahmed.com Installing Cookbook Gems:
    chefnode.ahmed.com Compiling Cookbooks...
    chefnode.ahmed.com [2016-06-08T03:05:08-07:00] WARN: Node chefnode.ahmed.com has an empty run list.
    chefnode.ahmed.com Converging 0 resources
    chefnode.ahmed.com
    chefnode.ahmed.com Running handlers:
    chefnode.ahmed.com Running handlers complete
    chefnode.ahmed.com Chef Client finished, 0/0 resources updated in 05 seconds
    chefnode.ahmed.com Hasta la vista, baby

We can check the node using the below command.

    [root@chefworkstation chef-repo]# knife client list                          
    ahmedinc-validator
    chefnode.ahmed.com

Here is how we can see the new node on the `chef-server`.

![cookbook on chef-server](http://zubayr.github.io/images/chef_server_node_added.png)

   
Lets update the starter cookbook and upload to server and add the cookbook to the node.
Update `/home/ahmed/chef-repo/cookbooks/starter/recipes/default.rb` with below contents.

    log "Welcome to Chef, #{node["starter_name"]}!" do
      level :info
    end

    file '/etc/my_first_file' do
            content 'This is my first file creation using chef server'
    end

    file '/etc/my_second_file' do
            content 'My Second file'
    end

Here is the contents from the command output.

    [root@chefworkstation chef-repo]# pwd                                         
    /home/ahmed/chef-repo
    [root@chefworkstation chef-repo]# cat cookbooks/starter/recipes/default.rb
    # This is a Chef recipe file. It can be used to specify resources which will
    # apply configuration to a server.

    log "Welcome to Chef, #{node["starter_name"]}!" do
      level :info
    end

    file '/etc/my_first_file' do
            content 'This is my first file creation using chef server'
    end

    file '/etc/my_second_file' do
            content 'My Second file'
    end

Now lets upload the cookbook to the server.

    [root@chefworkstation chef-repo]# knife upload cookbooks/starter
    Created cookbooks/starter
    [root@chefworkstation chef-repo]#


**Starter cookbook on the `chef-server`**

![cookbook on chef-server](http://zubayr.github.io/images/chef_server_cookbook_synced.png)

**Contents of `starter` Cookbook**

![cookbook on chef-server](http://zubayr.github.io/images/chef_server_cookbook_starter_contents.png)


Now lets configure the node to get the cookbook assigned.

**Edit Node Configuration**

![files](http://zubayr.github.io/images/chef_server_add_cookbook_edit.png)

**Select the cookbook and add to Current Run List**

![files](http://zubayr.github.io/images/chef_server_assign_cookbook_1.png)

![files](http://zubayr.github.io/images/chef_server_assign_cookbook_2.png)
    

## Sync `client` with `server` using `chef-client` command. 

Once we have assigned we can logon to the ched-node and execute `chef-client`. This will communicate to the chef-server and get the cookbook and syncronize the server.

    [root@chefnode Downloads]# chef-client
    Starting Chef Client, version 12.10.24
    resolving cookbooks for run list: ["starter"]
    Synchronizing Cookbooks:
      - starter (1.0.0)
    Installing Cookbook Gems:
    Compiling Cookbooks...
    Converging 2 resources
    Recipe: starter::default
      * log[Welcome to Chef, Sam Doe!] action write

      * file[/etc/my_first_file] action create
        - create new file /etc/my_first_file
        - update content in file /etc/my_first_file from none to ad169c
        --- /etc/my_first_file      2016-06-08 03:37:02.461008058 -0700
        +++ /etc/.chef-my_first_file20160608-3200-18m1pty   2016-06-08 03:37:02.460008058 -0700
        @@ -1 +1,2 @@
        +This is my first file creation using chef server
        - restore selinux security context

    Running handlers:
    Running handlers complete
    Chef Client finished, 2/2 resources updated in 05 seconds


## Getting repo from `git` and sync with `chef-server`.

On the workstation node.

	[root@chefworkstation cookbooks]# pwd
	/home/ahmed/chef-repo/cookbooks


Get `repo` from git and upload to chef server.

    [root@chefworkstation cookbooks]# git clone https://github.com/zubayr/cm_setup
    Initialized empty Git repository in /home/ahmed/chef-repo/cookbooks/cm_setup/.git/
    remote: Counting objects: 62, done.
    remote: Compressing objects: 100% (35/35), done.
    remote: Total 62 (delta 5), reused 61 (delta 4), pack-reused 0
    Unpacking objects: 100% (62/62), done.
    [root@chefworkstation cookbooks]# cd ..
    [root@chefworkstation chef-repo]# ls
    cookbooks  README.md  roles

Upload to server.

    [root@chefworkstation chef-repo]# knife upload cookbooks/cm_setup
    Created cookbooks/cm_setup
    [root@chefworkstation chef-repo]#

As seen on the server.

![files](http://zubayr.github.io/images/chef_server_cookbook_uploaded_to_server_git.png)
