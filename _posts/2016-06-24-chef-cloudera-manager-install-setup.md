---
title: Cloudera Manager Setup Using Chef [CentOS 6.6]
category: ['Centos', 'Chef', 'Chefdk', 'Cloudera-Manager', 'Cdh']
tags: ['centos', 'chef', 'chefdk', 'cloudera-manager', 'cdh']
---

This cookbook **[currently as of now]** can be used to setup a Cloudera Manager Server (Management Server) running on MySQL database.
But the intended use for this cookbook **[rather a wishlist]** is to do more. Simply put `Auto Deployment` of a Cloudera Hadoop Cluster using Chef, Python and Cloudera API. This will help create cluster for a `development/test/preproduction/production` environment on a click of a button.

* Attributes
* Recipe
* Usage

### Get the Cookbook.

Can be downloaded from the link. [Cloudera Manager Setup](https://github.com/zubayr/cm_setup/archive/version_0.1b.zip)


### How would the Setup Look like.

Nodes and the runlist which will be assigned.

* Cloudera Manager - Runlist  `cm_setup` default runlist which will include all the installations.
* All Other nodes - Runlist, will be assigning the Role `base_node_install` which we will create below.

#### What does `cm_setup` default cookbook have.

Common installations, like disable `selinux` and `iptables`.

    # Setting up commons
    include_recipe 'cm_setup::commons'

Setting up `sysctl.conf` configuration optimized for Hadoop.

    include_recipe 'cm_setup::sysctl_setup'

Installing and configuring `ntpd`

    include_recipe 'cm_setup::ntpd_setup'

Creating user(s) for `cloudera-manager`.

    include_recipe 'cm_setup::users_setup'

Creating `/etc/hosts` file as required by `Hadoop` cluster.

    include_recipe 'cm_setup::hostfile_setup'

Creating a `mysql` setup. Setting up `cloudera-manager` on `mysql`.

    include_recipe 'cm_setup::mysql_setup'
      include_recipe 'cm_setup::mysql_install'
      include_recipe 'cm_setup::mysql_configure'
      include_recipe 'cm_setup::mysql_user_setup

Installing `mysql_connector`.

    include_recipe 'cm_setup::mysql_connector_setup'

Installing `cloudera-daemons` and `agent`.

    include_recipe 'cm_setup::cloudera_install_setup'
      configuring database using the db script.
      Autostarting `cloudera-scm-server`.

#### What does the `base_node_install` Role have.

Common installations, like disable `selinux` and `iptables`.

    # Setting up commons
    include_recipe 'cm_setup::commons'

Setting up `sysctl.conf` configuration optimized for Hadoop.

    include_recipe 'cm_setup::sysctl_setup'

Installing and configuring `ntpd`

    include_recipe 'cm_setup::ntpd_setup'

Creating user(s) for `cloudera-manager`.

    include_recipe 'cm_setup::users_setup'

Creating `/etc/hosts` file as required by `Hadoop` cluster.

    include_recipe 'cm_setup::hostfile_setup'



### Role.

To setup non-mgmt nodes we can create a role and assign the nodes this role, so that the base setup on that node is completed.
Below is a JSON for the Role for base setup.

    {
       "name": "base_node_install",
       "description": "Base Installation for Node other than the Clouder Manager Node.",
       "json_class": "Chef::Role",
       "default_attributes": {

       },
       "override_attributes": {

       },
       "chef_type": "role",
       "run_list": [
         "recipe[cm_setup::commons]",
         "recipe[cm_setup::sysctl_setup]",
         "recipe[cm_setup::ntpd_setup]",
         "recipe[cm_setup::users_setup]",
         "recipe[cm_setup::hostfile_setup]"
       ],
       "env_run_lists": {

       }
    }

Creating `role` on the Chef Server.

    ┌─[ahmed][zubair-HP-ProBook][±][master U:2 ✗][~/work/chef-repo]
    └─▪ knife role create base_node_install

Add the contents above to the `role`. Once we are done then we can list then using below command.

    ┌─[ahmed][zubair-HP-ProBook][±][master U:2 ✗][~/work/chef-repo]
    └─▪ knife role list
    base_node_install
    testrole

Now we add the role to each on the nodes which act as a managed node like a namenode, standbynamenode, resourcemanager etc.

#### Before we assign the Role.

Before we assign the role, we need to bootstrap the node so that it is added to the Chef Server.
Below is the command to add the node to the Chef Server.

    knife bootstrap <host_ip_address> --ssh-port <ssh_port> --ssh-user <username> --ssh-password <password> --sudo

NOTE : The user we use should have `sudo` privileges so that chef can install the client on the node.
Here is the output for the vagrant node which was used to test the Cookbook.

    ┌─[ahmed][zubair-HP-ProBook][±][master ✓][~/work/chef-repo/cookbooks/cm_setup]
    └─▪ knife bootstrap 127.0.0.1 --ssh-port 2222 --ssh-user vagrant --ssh-password vagrant --sudo
    Doing old-style registration with the validation key at /home/ahmed/work/chef-repo/.chef/happy-minds-validator.pem...
    Delete your validation key in order to use your user credentials instead

    Connecting to 127.0.0.1
    127.0.0.1 -----> Installing Chef Omnibus (-v 12)
    127.0.0.1 downloading https://omnitruck-direct.chef.io/chef/install.sh
    127.0.0.1   to file /tmp/install.sh.3341/install.sh
    127.0.0.1 trying wget...
    127.0.0.1 el 6 x86_64
    127.0.0.1 Getting information for chef stable 12 for el...
    127.0.0.1 downloading https://omnitruck-direct.chef.io/stable/chef/metadata?v=12&p=el&pv=6&m=x86_64
    127.0.0.1   to file /tmp/install.sh.3361/metadata.txt
    127.0.0.1 trying wget...
    127.0.0.1 sha1	44e71beed0cc0db2481c3e3d2108ad218c32dade
    127.0.0.1 sha256	e51559dc7747c03b446f9d1a3cdbb122f274352ba0ed7dd8fdac41e10514b9e2
    127.0.0.1 url	https://packages.chef.io/stable/el/6/chef-12.11.18-1.el6.x86_64.rpm
    127.0.0.1 version	12.11.18
    127.0.0.1 downloaded metadata file looks valid...
    127.0.0.1 downloading https://packages.chef.io/stable/el/6/chef-12.11.18-1.el6.x86_64.rpm
    127.0.0.1   to file /tmp/install.sh.3361/chef-12.11.18-1.el6.x86_64.rpm
    127.0.0.1 trying wget...
    127.0.0.1 Comparing checksum with sha256sum...
    127.0.0.1 Installing chef 12
    127.0.0.1 installing with rpm...
    127.0.0.1 warning: /tmp/install.sh.3361/chef-12.11.18-1.el6.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID 83ef826a: NOKEY
    127.0.0.1 Preparing...                                                            (1########################################### [100%]
    127.0.0.1    1:chef                                                               ( ########################################### [100%]
    127.0.0.1 Thank you for installing Chef!
    127.0.0.1 Starting the first Chef Client run...
    127.0.0.1 Starting Chef Client, version 12.11.18
    127.0.0.1 Creating a new client identity for localhost.localdomain using the validator key.
    127.0.0.1 resolving cookbooks for run list: []
    127.0.0.1 Synchronizing Cookbooks:
    127.0.0.1 Installing Cookbook Gems:
    127.0.0.1 Compiling Cookbooks...
    127.0.0.1 [2016-06-24T15:25:45+02:00] WARN: Node localhost.localdomain has an empty run list.
    127.0.0.1 Converging 0 resources
    127.0.0.1
    127.0.0.1 Running handlers:
    127.0.0.1 Running handlers complete
    127.0.0.1 Chef Client finished, 0/0 resources updated in 09 seconds


#### Logon to Chef Server and Edit Run List.

![Chef Server and Edit Run List](https://zubayr.github.io/images/chef_edit_runlist.png)

#### Select Role to be Assigned.

![Select Role to be Assigned](https://zubayr.github.io/images/chef_select_role_to_assign.png)

#### Assigned Role and Save.

![Assigned Role and Save](https://zubayr.github.io/images/chef_assign_role_and_save.png)

#### Chef Role is Assigned.

![Chef Role is Assigned](https://zubayr.github.io/images/chef_role_assigned.png)

#### Executing `sudo chef-client` on Node.

![Executing sudo chef-client on Node](https://zubayr.github.io/images/chef_client_execute_on_the_node.png)




### Attributes.

Below are the set of attributes which can be changed as per requirement.

    ┌─[ahmed][zubair-HP-ProBook][±][master ✓][~/work/chef-repo/cookbooks/cm_setup/attributes]
    └─▪ tree
    .
    |____hosts_attr.rb
    |____default.rb
    |____mysql_attr.rb
    |____sudo_attr.rb
    |____cdh_attr.rb
    |____sysctl_attr.rb
    |____security_sssd_attr.rb
    |____security_krb5_attr.rb
    |____ntp_attr.rb

#### `hosts_attr` File

This file has the host information which need to be populated in the `/etc/hosts` file.

`hostsfile` cookbook [https://github.com/customink-webops/hostsfile](https://github.com/customink-webops/hostsfile).

    ┌─[ahmed][zubair-HP-ProBook][±][master ✓][~/work/chef-repo/cookbooks/cm_setup/attributes]
    └─▪ cat hosts_attr.rb

    #
    # Server informatoin for the `/etc/hosts` file changes this as required
    #

    default['etc_hosts_entries']['9.1.1.1']['hostname'] = 'server9.ahmed.com'
    default['etc_hosts_entries']['9.1.1.1']['aliases']  = ['server9']
    default['etc_hosts_entries']['9.1.1.1']['comment']  = 'Server9'
    default['etc_hosts_entries']['9.1.1.1']['action']   = :create_if_missing

#### `mysql_attr` File.

This has parameters related to `mysql` more attributes can be twicked more information can be found on the base cookbook `mysql`, `mysql_connector`, `database`.

`mysql` cookbook for creating the mysql instance. `mysql_connector` cookbook for creating the connector. `database` cookbook to create database and database users.

* `mysql` Base cookbook [https://github.com/chef-cookbooks/mysql](https://github.com/chef-cookbooks/mysql).
* `mysql_connector` Base cookbook [https://supermarket.chef.io/cookbooks/mysql_connector](https://supermarket.chef.io/cookbooks/mysql_connector).
* `database` Base cookbook [https://github.com/chef-cookbooks/database](https://github.com/chef-cookbooks/database).

File here.

    ┌─[ahmed][zubair-HP-ProBook][±][master ✓][~/work/chef-repo/cookbooks/cm_setup/attributes]
    └─▪ cat mysql_attr.rb
    #
    # MySQL connector attributes
    # => https://supermarket.chef.io/cookbooks/mysql_connector
    # => https://supermarket.chef.io/cookbooks/mysql_connector/download
    #

    default['mysql_connector']['j']['install_paths'] = ['/usr/share/java']
    default['mysql_connector']['j']['version'] = '5.1.36'

    #
    # MySQL user, Configuration and services
    #
    # #
    # # Installing `mysqld`
    # # => https://github.com/chef-cookbooks/mysql
    # # => https://supermarket.chef.io/cookbooks/mysql#knife
    # #
    #
    # #
    # # Setting up user for the mysql database.
    # # => https://github.com/chef-cookbooks/database
    # # => https://supermarket.chef.io/cookbooks/database#knife
    # #
    #
    #

    default['mysql']['configuring']['database_service_name'] = 'default'
    default['mysql']['configuring']['database_name'] = 'cmdb'
    default['mysql']['configuring']['database_root_password'] = 'root@123'


    default['mysql']['configuring']['database_user'] = 'cmadmin'
    default['mysql']['configuring']['database_password'] = 'cmadmin@123'
    default['mysql']['configuring']['database_user_privileges'] = [:all]
    default['mysql']['configuring']['database_user_privileges_host'] = '%'

    default['mysql']['configuring']['host_ip'] = '127.0.0.1'
    default['mysql']['configuring']['port'] = '3306'
    default['mysql']['configuring']['version'] = '5.5'

#### `hostsfile` File

Creating `users` and `sudo` users on the server.

* `users` cookbook [https://github.com/chef-cookbooks/users](https://github.com/chef-cookbooks/users).
* `sudo` cookbook [https://github.com/chef-cookbooks/users](https://github.com/chef-cookbooks/users).

Config File.

    ┌─[ahmed][zubair-HP-ProBook][±][master U:3 ✗][~/work/chef-repo/cookbooks/cm_setup/attributes]
    └─▪ cat sudo_attr.rb
    #
    # Adding sudo attributes
    #

    #
    # User Setup
    #
    # #
    # # Creating a admin user/group for clouderamanager
    # # https://github.com/chef-cookbooks/users
    # # https://supermarket.chef.io/cookbooks/users#knife
    #

    default['users_setup']['groups'] = { 'sysadmin' => 2300, 'cmadmin' => 2301 }

    #
    # Creating sudo users
    # => https://supermarket.chef.io/cookbooks/sudo#knife
    # => https://github.com/chef-cookbooks/sudo
    #

    default['authorization']['sudo']['groups'] = ['cmadmin', 'sysadmin']
    default['authorization']['sudo']['users'] = ['cmadmin', 'vagrant', 'sysadminuser']
    default['authorization']['sudo']['passwordless'] = true

#### `cdh_attr.rb` File

Here we can configure information related to `cdh`.

* Creating Repository
* Installation Packages
* Services

`yum` cookbook [https://github.com/chef-cookbooks/yum/](https://github.com/chef-cookbooks/yum/)

    ┌─[ahmed][zubair-HP-ProBook][±][master U:5 ✗][~/work/chef-repo/cookbooks/cm_setup/attributes]
    └─▪ cat cdh_attr.rb

    #
    # Cloudera Manager installation and services
    #

    default['cdh_install']['install_packages'] = [
                                                  'oracle-j2sdk1.7',
                                                  'cloudera-manager-daemons',
                                                  'cloudera-manager-server'
                                                ]
    default['cdh_install']['cm_services'] = [
                                              'cloudera-scm-server'
                                            ]


    #
    # Repository Configuration
    #
    # # Setting up repos
    # # => https://supermarket.chef.io/cookbooks/yum
    # # => https://github.com/chef-cookbooks/yum/
    # #
    #

    # description 'Extra Packages for Enterprise Linux'
    # mirrorlist 'http://mirrors.fedoraproject.org/mirrorlist?repo=epel-6&arch=$basearch'
    # gpgkey 'http://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-6'

    default['yum_repository']['epel']['description'] = 'Extra Packages for Enterprise Linux'
    default['yum_repository']['epel']['mirrorlist'] = 'http://mirrors.fedoraproject.org/mirrorlist?repo=epel-6&arch=$basearch'
    default['yum_repository']['epel']['gpgkey'] = 'http://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-6'


    # description 'Packages for Cloudera Manager, Version 5, on RedHat or CentOS 6 x86_64 '
    # baseurl 'https://archive.cloudera.com/cm5/redhat/6/x86_64/cm/5/'
    # gpgkey 'https://archive.cloudera.com/cm5/redhat/6/x86_64/cm/RPM-GPG-KEY-cloudera'

    default['yum_repository']['cm']['description'] = 'Packages for Cloudera Manager, Version 5, on RedHat or CentOS 6 x86_64 '
    default['yum_repository']['cm']['baseurl'] = 'https://archive.cloudera.com/cm5/redhat/6/x86_64/cm/5/'
    default['yum_repository']['cm']['gpgkey'] = 'https://archive.cloudera.com/cm5/redhat/6/x86_64/cm/RPM-GPG-KEY-cloudera'

#### `sysctl_attr` File.

This file is to update `sysctl.conf`. All attributes are from the `sysctl` cookbook.

`sysctl` cookbook [https://github.com/svanzoest-cookbooks/sysctl](https://github.com/svanzoest-cookbooks/sysctl)

    ┌─[ahmed][zubair-HP-ProBook][±][master U:5 ✗][~/work/chef-repo/cookbooks/cm_setup/attributes]
    └─▪ cat sysctl_attr.rb

    #
    # Setting up custom sysctl configuration
    # TODO: We need to make the parameter to be added from attributes.
    #
    # template '/etc/sysctl.conf' do
    #   source 'sysctl.conf.erb'
    # end
    #
    # Setting up sysctl.conf
    # => https://supermarket.chef.io/cookbooks/sysctl
    # => https://github.com/svanzoest-cookbooks/sysctl
    #

#### `security_sssd_attr` File.

This file is to setup (install and configure) `sssd` on the node.

`sysctl_ldap` cookbook [https://github.com/tas50/chef-sssd_ldap](https://github.com/tas50/chef-sssd_ldap)


    ┌─[ahmed][zubair-HP-ProBook][±][master U:5 ✗][~/work/chef-repo/cookbooks/cm_setup/attributes]
    └─▪ cat security_sssd_attr.rb
    #
    # SSSD installation and configuration
    #
    #

    #
    # Installing and configuring SSSD
    # => https://supermarket.chef.io/cookbooks/sssd_ldap
    # => https://github.com/tas50/chef-sssd_ldap
    #

#### `security_krb5_attr` File.

This file attributes are to install and configure `krb5` for a node.

`krb5` cookbook [https://github.com/atomic-penguin/cookbook-krb5](https://github.com/atomic-penguin/cookbook-krb5)


    ┌─[ahmed][zubair-HP-ProBook][±][master U:5 ✗][~/work/chef-repo/cookbooks/cm_setup/attributes]
    └─▪ cat security_krb5_attr.rb
    #
    # krb5 installation and configuration
    #


    #
    # Installing and configuring krb5
    # => https://supermarket.chef.io/cookbooks/krb5
    # => https://github.com/atomic-penguin/cookbook-krb5
    #

#### `ntp_attr` File.

Setting up `ntp` on a node.

    ┌─[ahmed][zubair-HP-ProBook][±][master U:5 ✗][~/work/chef-repo/cookbooks/cm_setup/attributes]
    └─▪ cat ntp_attr.rb
    #
    # configuring servers
    # => https://supermarket.chef.io/cookbooks/ntpd#knife
    # => https://github.com/rogerdelph/cookbook-ntpd
    #

    default['ntp']['mode_servers'] = ['0.pool.ntp.org', '1.pool.ntp.org']


### `default` Recipe Details.

Common installations, like disable `selinux` and `iptables`.

    # Setting up commons
    include_recipe 'cm_setup::commons'

Setting up `sysctl.conf` configuration optimized for Hadoop.

    include_recipe 'cm_setup::sysctl_setup'

Installing and configuring `ntpd`

    include_recipe 'cm_setup::ntpd_setup'

Creating user(s) for `cloudera-manager`.

    include_recipe 'cm_setup::users_setup'

Creating `/etc/hosts` file as required by `Hadoop` cluster.

    include_recipe 'cm_setup::hostfile_setup'

Creating a `mysql` setup. Setting up `cloudera-manager` on `mysql`.

    include_recipe 'cm_setup::mysql_setup'
      include_recipe 'cm_setup::mysql_install'
      include_recipe 'cm_setup::mysql_configure'
      include_recipe 'cm_setup::mysql_user_setup

Installing `mysql_connector`.

    include_recipe 'cm_setup::mysql_connector_setup'

Installing `cloudera-daemons` and `agent`.

    include_recipe 'cm_setup::cloudera_install_setup'
      configuring database using the db script.
      Autostarting `cloudera-scm-server`.

Installation and Configuration of `sssd`. [ Unit Test complete - Need to do TEST on live environment ]

Installation and Configuration of `krb5`. [ Unit Test complete - Need to do TEST on live environment ]

Configuration of Cloudera Using Cloudera API. [TODO]  


### Usage.

Below are the steps to setup and environment to execute this cookbook.

#### Update the `.kitchen.yml` file with below content [ if required - OPTIONAL ]

File can be found in the `${CHEF_COOKBOOK_HOME}/.kitchen.yml`.

    ┌─[ahmed][zubair-HP-ProBook][±][master ✓][~/work/chef-repo/cookbooks/cm_setup]
    └─▪ cat .kitchen.yml
    ---
    driver:
      name: vagrant

    provisioner:
      name: chef_zero

    # Uncomment the following verifier to leverage Inspec instead of Busser (the
    # default verifier)
    # verifier:
    #   name: inspec

    platforms:
      - name: grtjn/centos-6.5

    suites:
      - name: default
        run_list:
          - recipe[cm_setup::default]
        attributes:


#### Check the for the vagrant box which will be used.

Command

    kitchen list

Output

    ┌─[ahmed][zubair-HP-ProBook][±][master ✓][~/work/chef-repo/cookbooks/starter]
    └─▪ kitchen list
    Instance                 Driver   Provisioner  Verifier  Transport  Last Action
    default-grtjn-centos-65  Vagrant  ChefSolo     Busser    Ssh        <Not Created>


#### `create` node.

Command

    kitchen create

Output

    ┌─[ahmed][zubair-HP-ProBook][±][master ↑1 U:1 ?:3 ✗][~/work/chef-repo/cookbooks/cm_setup]
    └─▪ kitchen create
    -----> Starting Kitchen (v1.8.0)
    -----> Creating <default-grtjn-centos-65>...
           Bringing machine 'default' up with 'virtualbox' provider...
           ==> default: Importing base box 'grtjn/centos-6.5'...
    ==> default: Matching MAC address for NAT networking...
           ==> default: Checking if box 'grtjn/centos-6.5' is up to date...
           ==> default: Setting the name of the VM: kitchen-starter-default-grtjn-centos-65_default_1466270503111_60773
           ==> default: Fixed port collision for 22 => 2222. Now on port 2200.
           ==> default: Clearing any previously set network interfaces...
           ==> default: Preparing network interfaces based on configuration...
               default: Adapter 1: nat
           ==> default: Forwarding ports...
               default: 22 (guest) => 2200 (host) (adapter 1)
           ==> default: Booting VM...
           ==> default: Waiting for machine to boot. This may take a few minutes...
               default: SSH address: 127.0.0.1:2200
               default: SSH username: vagrant
               default: SSH auth method: private key
           ==> default: Machine booted and ready!
           ==> default: Checking for guest additions in VM...
               default: The guest additions on this VM do not match the installed version of
               default: VirtualBox! In most cases this is fine, but in rare cases it can
               default: prevent things such as shared folders from working properly. If you see
               default: shared folder errors, please make sure the guest additions within the
               default: virtual machine match the version of VirtualBox you have installed on
               default: your host and reload your VM.
               default:
               default: Guest Additions Version: 4.3.8
               default: VirtualBox Version: 5.0
           ==> default: Setting hostname...
           ==> default: Machine not provisioned because `--no-provision` is specified.
           [SSH] Established
           Vagrant instance <default-grtjn-centos-65> created.
           Finished creating <default-grtjn-centos-65> (0m52.53s).
    -----> Kitchen is finished. (0m52.65s)

#### Login to the `node`.

Command

    kitchen login

Output

    ┌─[ahmed][zubair-HP-ProBook][±][master ↑1 U:1 ?:3 ✗][~/work/chef-repo/cookbooks/cm_setup]
    └─▪ kitchen login
    Last login: Sat Jun 18 17:22:13 2016 from 10.0.2.2
    [vagrant@default-grtjn-centos-65 ~]$ cat /etc/hosts
    127.0.0.1   default-grtjn-centos-65 localhost localhost.localdomain localhost4 localhost4.localdomain4
    ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
    [vagrant@default-grtjn-centos-65 ~]$ exit
    logout
    Connection to 127.0.0.1 closed.


#### `converge` - Cookbook with the node.

Command

    kitchen converge

Output

    ┌─[ahmed][zubair-HP-ProBook][±][master ↑1 U:2 ✗][~/work/chef-repo/cookbooks/cm_setup]
    └─▪ kitchen converge
    -----> Starting Kitchen (v1.8.0)
    -----> Converging <default-grtjn-centos-65>...
           Preparing files for transfer
           Preparing dna.json
           Resolving cookbook dependencies with Berkshelf 4.3.3...
           Removing non-cookbook files before transfer
           Preparing data_bags
           Preparing validation.pem
           Preparing client.rb
    -----> Installing Chef Omnibus (install only if missing)
           Downloading https://www.chef.io/chef/install.sh to file /tmp/install.sh
           Trying wget...
           Trying curl...
           Download complete.
           el 6 x86_64
           Getting information for chef stable  for el...
           downloading https://omnitruck-direct.chef.io/stable/chef/metadata?v=&p=el&pv=6&m=x86_64
             to file /tmp/install.sh.1983/metadata.txt
           trying wget...
           sha1 44e71beed0cc0db2481c3e3d2108ad218c32dade
           sha256 e51559dc7747c03b446f9d1a3cdbb122f274352ba0ed7dd8fdac41e10514b9e2
           url  https://packages.chef.io/stable/el/6/chef-12.11.18-1.el6.x86_64.rpm
           version  12.11.18
           downloaded metadata file looks valid...
           downloading https://packages.chef.io/stable/el/6/chef-12.11.18-1.el6.x86_64.rpm
             to file /tmp/install.sh.1983/chef-12.11.18-1.el6.x86_64.rpm
           trying wget...
           trying curl...
           Comparing checksum with sha256sum...

           WARNING WARNING WARNING WARNING WARNING WARNING WARNING WARNING WARNING

           You are installing an omnibus package without a version pin.  If you are installing
           on production servers via an automated process this is DANGEROUS and you will
           be upgraded without warning on new releases, even to new major releases.
           Letting the version float is only appropriate in desktop, test, development or
           CI/CD environments.

           WARNING WARNING WARNING WARNING WARNING WARNING WARNING WARNING WARNING

           Installing chef
           installing with rpm...
           warning: /tmp/install.sh.1983/chef-12.11.18-1.el6.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID 83ef826a: NOKEY
           Preparing...                                                            (100%########################################### [100%]
              1:chef                                                               (  1%########################################### [100%]
           Thank you for installing Chef!
           Transferring files to <default-grtjn-centos-65>
           Starting Chef Client, version 12.11.18
           Creating a new client identity for default-grtjn-centos-65 using the validator key.
           resolving cookbooks for run list: ["cm_setup::default"]
           Synchronizing Cookbooks:
             - hostsfile (2.4.5)
             - sudo (2.9.0)
             - users (2.0.3)
             - cm_setup (0.1.0)
             - sysctl (0.7.5)
             - mysql (7.1.1)
             - yum (3.11.0)
             - smf (2.2.8)
             - ohai (3.0.1)
             - database (5.1.2)
             - build-essential (6.0.0)
             - rbac (1.0.3)
             - openssl (4.4.0)
             - yum-mysql-community (0.2.0)
             - apt (4.0.0)
             - chef-sugar (3.3.0)
             - mingw (1.2.0)
             - seven_zip (2.0.1)
             - postgresql (4.0.6)
             - compat_resource (12.10.6)
             - windows (1.43.0)
             - chef_handler (1.4.0)
           Installing Cookbook Gems:
           Compiling Cookbooks...
           [2016-06-18T17:40:28+00:00] WARN: Chef::Provider::AptRepository already exists!  Cannot create deprecation class for LWRP provider apt_repository from cookbook apt
           [2016-06-18T17:40:28+00:00] WARN: AptRepository already exists!  Deprecation class overwrites Custom resource apt_repository from cookbook apt
           [2016-06-18T17:40:28+00:00] WARN: Cloning resource attributes for hostsfile_entry[3.3.3.3] from prior resource (CHEF-3694)
           [2016-06-18T17:40:28+00:00] WARN: Previous hostsfile_entry[3.3.3.3]: /tmp/kitchen/cache/cookbooks/cm_setup/recipes/hostfile_setup.rb:27:in `from_file'
           [2016-06-18T17:40:28+00:00] WARN: Current  hostsfile_entry[3.3.3.3]: /tmp/kitchen/cache/cookbooks/cm_setup/recipes/hostfile_setup.rb:34:in `from_file'
           Converging 37 resources

           ##
           ###################### VERBOSE ########################
           ##

           Recipe: cm_setup::cloudera_install_setup
             * yum_package[oracle-j2sdk1.7] action install (up to date)
             * yum_package[cloudera-manager-daemons] action install (up to date)
             * yum_package[cloudera-manager-server] action install (up to date)
           Recipe: cm_setup::mysql_setup
             * mysql_service[default] action restart
               * service[default :restart mysql-default] action restart
                 - restart service service[default :restart mysql-default]


           Running handlers:
           Running handlers complete
           Chef Client finished, 20/106 resources updated in 07 minutes 05 seconds
           Finished converging <default-grtjn-centos-65> (7m25.80s).

Above we have no issues and the cookbook converged successfully.

    ┌─[ahmed][zubair-HP-ProBook][±][master ✓][~/work/chef-repo/cookbooks/cm_setup]
    └─▪ kitchen list
    Instance                 Driver   Provisioner  Verifier  Transport  Last Action
    default-grtjn-centos-65  Vagrant  ChefZero     Busser    Ssh        Converged

#### Server Spec Verification using Kitchen.

Command

    kitchen verify

Output.

    ┌─[ahmed][zubair-HP-ProBook][±][master U:1 ✗][~/work/chef-repo/cookbooks/cm_setup]
    └─▪ kitchen verify
    -----> Starting Kitchen (v1.8.0)
    -----> Verifying <default-grtjn-centos-65>...
           Preparing files for transfer
    -----> Busser installation detected (busser)
           Installing Busser plugins: busser-serverspec
           Plugin serverspec already installed
           Removing /tmp/verifier/suites/serverspec
           Transferring files to <default-grtjn-centos-65>
    -----> Running serverspec test suite
           /opt/chef/embedded/bin/ruby -I/tmp/verifier/suites/serverspec -I/tmp/verifier/gems/gems/rspec-support-3.4.1/lib:/tmp/verifier/gems/gems/rspec-core-3.4.4/lib /opt/chef/embedded/bin/rspec --pattern    /tmp/verifier/suites/serverspec/\*\*/\*_spec.rb --color --format documentation --default-path /tmp/verifier/suites/serverspec

           cm_setup::default
             File "/etc/yum.repos.d/cloudera-manager.repo"
               should exist
             File "/etc/mysql-default/conf.d/default.cnf"
               should exist
               should be file
               should contain "max_connections = 550"
             File "/etc/sysctl.d/99-chef-attributes.conf"
               should exist
               should be file
               should contain "vm.dirty_ratio"
               should contain "vm.swappiness"
               should contain "vm.nr_hugepages"
             Package "ntp"
               should be installed
             Package "oracle-j2sdk1.7"
               should be installed
             Package "cloudera-manager-daemons"
               should be installed
             Package "cloudera-manager-server"
               should be installed
             File "/etc/hosts"
               should exist
               should be file
               should contain "namenode.ahmed.com"
               should contain "standbynamenode.ahmed.com"
               should contain "resourcemanager.ahmed.com"
             File "/etc/cloudera-scm-server/db.properties"
               should exist
               should be file
               should contain "user=cmadmin"
               should contain "name=cmdb"
             User "cmadmin"
               should exist
               should have home directory "/home/cmadmin"
             User "sysadminuser"
               should exist
               should have home directory "/home/sysadminuser"
             MySQL config parameters
               Mysql config "innodb_flush_log_at_trx_commit"
                 value
                   example at /tmp/verifier/suites/serverspec/default_spec.rb:55
               Mysql config "socket"
                 value
                   example at /tmp/verifier/suites/serverspec/default_spec.rb:59
               Mysql config "innodb_flush_method"
                 value
                   example at /tmp/verifier/suites/serverspec/default_spec.rb:66
               Mysql config "innodb_log_file_size"
                 value
                   example at /tmp/verifier/suites/serverspec/default_spec.rb:70
             Yumrepo "epel"
               should exist
               should be enabled
             Yumrepo "cloudera-manager"
               should exist
               should be enabled

           Finished in 1.32 seconds (files took 0.52476 seconds to load)
           34 examples, 0 failures

           Finished verifying <default-grtjn-centos-65> (0m6.22s).
    -----> Kitchen is finished. (0m6.66s)

#### Cloudera Manager UI.

Logon to the node and open up a browser and hit.

    http://127.0.0.1:7180/

You will see the cloudera manager UI. **NOTE: This will take a while for the first time, as clouder will initialize the database for first time use.**   
