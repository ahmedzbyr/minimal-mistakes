---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: Encrypted Data Bags - Chef
category: ['Chef', 'Data-Bags', 'Linux', 'Ubuntu', 'Centos', 'Kitchen']
tags: ['chef', 'linux', 'ubuntu', 'centos', 'kitchen']
---
Data Bags are a way to store information on the `chef-server` which all the cookbooks can access.
Few more additional advantages are that we can `encrypt` the data-bags as well, this will help in keeping any sensitive information like user/password.

What we are doing now is to store user information in a data-bag and use them to create users on the servers.
We will be using `users` cookbook to create the users.

#### `users` cookbook details.

- [`https://github.com/chef-cookbooks/users`](https://github.com/chef-cookbooks/users)<br>
- [`https://supermarket.chef.io/cookbooks/users`](https://supermarket.chef.io/cookbooks/users)

### Steps.

1. Create a `secret` file.
2. Create `json` user files.
3. Create `data-bag` on the `chef-server`.
4. Add the local `user`.json file to `data-bag` with the `secret` file created.


### Creating `secret` file.

We will be using the `openssl` to create a random file.


`user` secret.

~~~ ruby
openssl rand -base64 512 > ~/work/chef-repo/cookbooks/init-setup/secret-files/user_data_bags_encrypted_secret
~~~

Here is how the file would look like.

~~~ ruby
┌─[ahmed][zubair-HP-ProBook][~]
└─▪ openssl rand -base64 512
whPbpd9VtLSYsasvnqdiSpWKsSB3hp8S3dwyNpEOBFFCjYQjvxe2DNJp2v4Ow2qM
4Y3TUSrXqSkiR7BqzdnTAi2wrUOaqjxCpf4yUdbkCaeEAFJpegkJ9gCYQvVXMH72
36YQZO+xbZUvSHMaWGnkh2H0KNqeKgk+ytCKotWX2FQkcCGtzbn8RMCWx60lswpc
ZGUN6kzV+jwDkyDyFPmKkheOM0wowcKrDMogyccsGpPE2v/j/nLp7YngxyxQboxh
eE+xXhKiyB4C4s3IkowUy03ucFNWanU2BWOkVqzJKDW6w5YZXFBuEe9awLgryj+8
FtROy1fttfsjR5zmnim3iU5XgdvQB2F3oof4cz83XVLJh+/HRhZc0FybKC0fO0dV
DVxJqIYIafrDTEEZPXXbDBSpPZoKNHQOUWT1y5av9tOMTFYofTSGlKqhy1bvGXiQ
Md7ra04m0abJ0xTarT++MfwkeKStlaM9LtTWE165FYmOtG91SEvp0iEnnN2+USIN
AVxuep4D3rE4hSazHKG3SDUy7GbQe69d57fLUSoHZc/RUYasVUQ8Glh+MmN6WqW6
YYdxktu2iWslYc3C+dNgy+6t5i9NnCon82xdXA25dC0JM7IYcEBn4FTqOx2pnN4m
hQx/5xJgZ3uI3BUMyTjCJotbwb1ut3GntHpkNb1ZHmg=
~~~

### Creating `user`.json files in `data_bags` directory.

We can create the json files anywhere we want, but will be using the default location for `data_bags`.

Creating directory for our data bag.

~~~ ruby
mkdir data_bags/user_data_bag
mkdir data_bags/root_data_bag
~~~

### Adding `json` files to our directory.

~~~ ruby
┌─[ahmed][zubair-HP-ProBook][±][master U:1 ✗][~/work/chef-repo/cookbooks/init-setup/data_bags]
└─▪ tree
.
|____user_data_bag
| |____sysadmin.json
|____root_data_bag
  |____root.json
~~~

#### We will be creating a user called `sysadmin`.

we are setting the password as `sysadmin@123`. We can generate our own using the below command.

~~~ ruby
┌─[ahmed][zubair-HP-ProBook][~]
└─▪ openssl passwd -1 'root@007'
$1$JbS7rQs0$dRIRoWJ7HIRIAftFoD/iF/
┌─[ahmed][zubair-HP-ProBook][~]
└─▪ openssl passwd -1 'sysadmin@123'
$1$Uat6dd6b$0FYr2a7NUpX8AnHQtaDwY/
┌─[ahmed][zubair-HP-ProBook][~]
└─▪
~~~

`json` file for `sysadmin`

~~~ json
{
     "id": "sysadmin",
     "password": "$1$Uat6dd6b$0FYr2a7NUpX8AnHQtaDwY/",
     "groups": [
       "sysadmin"
     ],
     "uid": 9000,
     "shell": "/bin/bash"
}
~~~

#### Update `root` password.

Here the `json` for root.

~~~ json
{
     "id": "root",
     "password": "$1$JbS7rQs0$dRIRoWJ7HIRIAftFoD/iF/",
     "uid": 0,
     "home" : "/root",
     "groups": ["root"],
     "action": "modify"
}
~~~

### Next we create the `data-bag` on the `chef-server`.

First we create the data-bag called `root_data_bag` and store the `root-user` information.
This is to make sure we separate the `root` user from the rest of the common users.

~~~ ruby
knife data bag create <data_bag_name>
knife data bag from file <data_bag_name> <path_to_json_file> --secret-file <path_to_secret_file>
~~~

Here is the command to create `root_data_bag`

~~~ ruby
knife data bag create root_data_bag
knife data bag from file root_data_bag data_bags/root_data_bag/root.json --secret-file secret-file/user_data_bags_encrypted_secret
~~~

Here is the command to create `user_data_bag`

~~~ ruby
knife data bag create user_data_bag
knife data bag from file user_data_bag data_bags/user_data_bag/sysadmin.json --secret-file secret-file/user_data_bags_encrypted_secret
~~~


### [TESTING] Setting up test environment, `.kitchen.yml` file.

Above is to run using the `chef-server`, if we want to test the setup using `kitchen`.

1. Get the encrypted file from the `chef-server`.
2. Create a new directory to store the encrypted data bags.
3. Update `.kitchen.yml` file.

#### First we can get the data from the server using the `--secret-file` to verify the contents.

Command to get the Information.

~~~ ruby
knife data bag show --secret-file <path_to_secret_file> <data_bag_name> <user_info>
~~~

Here is the output.

~~~ ruby
┌─[ahmed][zubair-HP-ProBook][±][master U:1 ?:4 ✗][~/work/chef-repo/cookbooks/init-setup]
└─▪ knife data bag show --secret-file secret-file/user_data_bags_encrypted_secret user_data_bag sysadmin
Encrypted data bag detected, decrypting with provided secret.
groups:   sysadmin
id:       sysadmin
password: $1$crcL.lu/$uIR/GRpX7aMnI2wUTT31S0
shell:    /bin/bash
uid:      9000
~~~

#### Next we get the encrypted data from the chef-server.

Use the command below.

~~~ ruby
knife data bag show user_data_bag sysadmin -Fj
~~~

Here is the output of the file.

~~~ ruby
┌─[ahmed][zubair-HP-ProBook][±][master U:1 ?:4 ✗][~/work/chef-repo/cookbooks/init-setup]
└─▪ knife data bag show user_data_bag sysadmin -Fj
WARNING: Encrypted data bag detected, but no secret provided for decoding. Displaying encrypted data.
{
  "id": "sysadmin",
  "password": {
    "encrypted_data": "QEQV2MBkDh6FOlO29vJgdgoM1kNH6xNkfBrB2K8E9WcfHOYdkHzZUuu0lMJU\nbqSW5TaWiW60gP3Xcgn/jOxVnw==\n",
    "iv": "Oro4TJcXRwbXAb8tG+3eJQ==\n",
    "version": 1,
    "cipher": "aes-256-cbc"
  },
  "groups": {
    "encrypted_data": "qEws3EZsvgzYJmRpzFTEQQJAZcYHLkbzYpwzZyGZbT0=\n",
    "iv": "usca8tkD0/tatXxX17KAKQ==\n",
    "version": 1,
    "cipher": "aes-256-cbc"
  },
  "uid": {
    "encrypted_data": "leILI+0wFS258IXf5UuNMBh+ZhKW+hJiQ0mtsW2a9gg=\n",
    "iv": "AiWB2YnkGHIkZzgivkmfjA==\n",
    "version": 1,
    "cipher": "aes-256-cbc"
  },
  "shell": {
    "encrypted_data": "61OW+eH8dynbXuL/HxWWuYHIJzd8ODB0H/MXA9tM69A=\n",
    "iv": "5SJcvDoBZdto5p2HKerUkg==\n",
    "version": 1,
    "cipher": "aes-256-cbc"
  }
}
~~~

- Lets create a directory to store our encrypted data bag. `mkdir ${COOKBOOK_HOME}/testing_encrypted_data_bags/data_bags/user_data_bag`
- Now copy this above contents (which is encrypted) into a directory created above.

~~~ ruby
vim ${COOKBOOK_HOME}/testing_encrypted_data_bags/data_bags/user_data_bag/sysadmin.json
~~~

#### Adding below lines to your `.kitchen.yml` file.

~~~ ruby
data_bags_path: '/home/ahmed/work/chef-repo/cookbooks/init-setup/testing_encrypted_data_bags/data_bags'
encrypted_data_bag_secret_key_path: "/home/ahmed/work/chef-repo/cookbooks/init-setup/secret-file/user_data_bags_encrypted_secret"
~~~

Here is the complete `yml` file.

~~~ yaml
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
  - name: ubuntu/trusty64

suites:
  - name: default
    data_bags_path: '/home/ahmed/work/chef-repo/cookbooks/init-setup/testing_encrypted_data_bags/data_bags'
    encrypted_data_bag_secret_key_path: "/home/ahmed/work/chef-repo/cookbooks/init-setup/secret-file/user_data_bags_encrypted_secret"
    run_list:
    - recipe[init-setup::default]
    attributes:
~~~

#### Commands to test using `kitchen`.

Creating the VM.

~~~ ruby
┌─[ahmed][zubair-HP-ProBook][±][master U:3 ?:1 ✗][~/work/chef-repo/cookbooks/init-setup]
└─▪ kitchen list
Instance                 Driver   Provisioner  Verifier  Transport  Last Action
default-ubuntu-trusty64  Vagrant  ChefZero     Busser    Ssh        <Not Created>
┌─[ahmed][zubair-HP-ProBook][±][master U:3 ?:1 ✗][~/work/chef-repo/cookbooks/init-setup]
└─▪ kitchen create
-----> Starting Kitchen (v1.8.0)
-----> Creating <default-ubuntu-trusty64>...
       Bringing machine 'default' up with 'virtualbox' provider...
       ==> default: Checking if box 'ubuntu/trusty64' is up to date...
       ==> default: A newer version of the box 'ubuntu/trusty64' is available! You currently
       ==> default: have version '20160824.1.0'. The latest is version '20160830.0.0'. Run
       ==> default: `vagrant box update` to update.
       ==> default: VirtualBox VM is already running.
       [SSH] Established
       Vagrant instance <default-ubuntu-trusty64> created.
       Finished creating <default-ubuntu-trusty64> (0m15.68s).
-----> Kitchen is finished. (0m16.03s)
┌─[ahmed][zubair-HP-ProBook][±][master U:3 ?:1 ✗][~/work/chef-repo/cookbooks/init-setup]
└─▪ kitchen list
Instance                 Driver   Provisioner  Verifier  Transport  Last Action
default-ubuntu-trusty64  Vagrant  ChefZero     Busser    Ssh        Created
~~~

Converging VM.

~~~ ruby
┌─[ahmed][zubair-HP-ProBook][±][master U:3 ?:1 ✗][~/work/chef-repo/cookbooks/init-setup]
└─▪ kitchen converge
-----> Starting Kitchen (v1.8.0)
-----> Converging <default-ubuntu-trusty64>...
       Preparing files for transfer
       Preparing dna.json
       Resolving cookbook dependencies with Berkshelf 4.3.3...
       Removing non-cookbook files before transfer
       Preparing data_bags
       Preparing secret
       Preparing validation.pem
       Preparing client.rb
-----> Chef Omnibus installation detected (install only if missing)
       Transferring files to <default-ubuntu-trusty64>
       Starting Chef Client, version 12.13.37
       Creating a new client identity for default-ubuntu-trusty64 using the validator key.
       resolving cookbooks for run list: ["init-setup::default"]
       Synchronizing Cookbooks:
         - init-setup (0.1.3)
         - chef-client (5.0.0)
         - users (3.0.0)
         - ntp (2.0.2)
         - apt-upgrade-once (0.2.1)
         - openssh (2.0.0)
         - sudo (2.11.0)
         - openssl-source (1.0.4)
         - nrpe (1.6.2)
         - cron (1.7.6)
         - logrotate (2.1.0)
         - iptables (2.2.0)
         - build-essential (6.0.4)
         - windows (1.44.3)
         - yum-epel (0.7.1)
         - compat_resource (12.14.0)
         - seven_zip (2.0.2)
         - chef_handler (1.4.0)
         - mingw (1.2.4)
         - yum (3.12.0)
       Installing Cookbook Gems:
       Compiling Cookbooks...
       /tmp/kitchen/cache/cookbooks/compat_resource/files/lib/chef_compat/monkeypatches/chef/run_context.rb:643: warning: already initialized constant Chef::RunContext::ChildRunContext::CHILD_STATE
       /opt/chef/embedded/lib/ruby/gems/2.1.0/gems/chef-12.13.37/lib/chef/run_context.rb:634: warning: previous definition of CHILD_STATE was here
       /tmp/kitchen/cache/cookbooks/compat_resource/libraries/chef_upstream_version.rb:2: warning: already initialized constant ChefCompat::CHEF_UPSTREAM_VERSION
       /tmp/kitchen/cache/cookbooks/compat_resource/libraries/chef_upstream_version.rb:2: warning: previous definition of CHEF_UPSTREAM_VERSION was here
       Converging 58 resources
       Recipe: init-setup::default
         * log[Welcome to Chef, Team!] action write

         * log[=============================] action write

         * log[Welcome to Server : default-ubuntu-trusty64] action write

         * log[=============================] action write

         * log[Server Hostname : default-ubuntu-trusty64 ] action write

         * log[Server Platform : ubuntu ] action write

         * log[Server IP Address : 10.0.2.15 ] action write

         * log[Server MAC Address : 08:00:27:1A:E9:1A ] action write

         * log[Server Recipes : ["init-setup::default"] ] action write

         * log[Server Roles : [] ] action write

         * log[Server OHAI Time : 1473215635.1009629 ] action write

         * users_manage[sysadmin] action create
           * group[sysadmin] action create (skipped due to only_if)
           * user[sysadmin] action create (up to date)
           * directory[/home/sysadmin/.ssh] action create (skipped due to only_if)
           * template[/home/sysadmin/.ssh/authorized_keys] action create (skipped due to only_if)
           * group[sysadmin] action create (up to date)
            (up to date)
         * users_manage[root] action create
           * group[root] action create (skipped due to only_if)
           * user[root] action modify (up to date)
           * directory[/root/.ssh] action create (skipped due to only_if)
           * template[/root/.ssh/authorized_keys] action create (skipped due to only_if)
           * group[root] action create (up to date)
            (up to date)
       Recipe: sudo::default
         * apt_package[sudo] action install (skipped due to not_if)
         * template[/etc/sudoers] action create (up to date)
       Recipe: apt-upgrade-once::default
         * execute[apt-update] action nothing (skipped due to action :nothing)
         * execute[apt-upgrade] action nothing (skipped due to action :nothing)
         * file[/etc/.apt-upgrade-run] action create (up to date)
       Recipe: openssh::default
         * apt_package[openssh-client] action install (up to date)
         * apt_package[openssh-server] action install (up to date)
         * template[/etc/ssh/ssh_config] action create (up to date)
         * template[/etc/ssh/sshd_config] action create (up to date)
         * execute[sshd-config-check] action nothing (skipped due to action :nothing)
         * service[ssh] action enable (up to date)
         * service[ssh] action start (up to date)
       Recipe: ntp::default
         * apt_package[ntp] action install (up to date)
         * apt_package[ntpdate] action install (up to date)
         * directory[/var/lib/ntp] action create (up to date)
         * directory[/var/log/ntpstats/] action create (up to date)
         * cookbook_file[/etc/ntp.leapseconds] action create (up to date)
       Recipe: ntp::apparmor
         * service[apparmor] action nothing (skipped due to action :nothing)
         * cookbook_file[/etc/apparmor.d/usr.sbin.ntpd] action create (up to date)
       Recipe: ntp::default
         * template[/etc/ntp.conf] action create (up to date)
         * execute[Force sync hardware clock with system clock] action run (skipped due to only_if)
         * service[ntp] action enable (up to date)
         * service[ntp] action start (up to date)
       Recipe: init-setup::default
         * apt_package[make] action install (up to date)
         * apt_package[gcc] action install (up to date)
         * apt_package[open-vm-tools] action install (up to date)
       Recipe: openssl-source::default
         * remote_file[/tmp/kitchen/cache/openssl-1.0.2f.tar.gz] action create (skipped due to not_if)
         * execute[unarchive_openssl] action nothing (skipped due to action :nothing)
         * execute[compile_openssl_source] action nothing (skipped due to action :nothing)
         * ruby_block[sync certificates] action nothing (skipped due to action :nothing)
         * execute[hash certificates with SHA1] action nothing (skipped due to action :nothing)
       Recipe: nrpe::_package_install
         * apt_package[nagios-nrpe-server] action install (up to date)
         * apt_package[nagios-plugins] action install (up to date)
         * apt_package[nagios-plugins-basic] action install (up to date)
         * apt_package[nagios-plugins-standard] action install (up to date)
       Recipe: nrpe::configure
         * directory[/etc/nagios/nrpe.d] action create (up to date)
         * template[/etc/nagios/nrpe.cfg] action create (up to date)
         * execute[nrpe-reload-systemd] action nothing (skipped due to action :nothing)
         * template[/lib/systemd/system/nrpe.service] action create (skipped due to only_if)
         * service[nagios-nrpe-server] action start (up to date)
         * service[nagios-nrpe-server] action enable (up to date)
         * ruby_block[updating of the list of checks] action run
           - execute the ruby block updating of the list of checks
       Recipe: init-setup::default
         * nrpe_check[check_users] action add
           * file[/etc/nagios/nrpe.d/check_users.cfg] action create (up to date)
            (up to date)
         * nrpe_check[check_load] action add
           * file[/etc/nagios/nrpe.d/check_load.cfg] action create (up to date)
            (up to date)
         * nrpe_check[check_hda1] action add
           * file[/etc/nagios/nrpe.d/check_hda1.cfg] action create (up to date)
            (up to date)
         * nrpe_check[check_zombie_procs] action add
           * file[/etc/nagios/nrpe.d/check_zombie_procs.cfg] action create (up to date)
            (up to date)
         * nrpe_check[check_total_procs] action add
           * file[/etc/nagios/nrpe.d/check_total_procs.cfg] action create (up to date)
            (up to date)
         * nrpe_check[check_root] action add
           * file[/etc/nagios/nrpe.d/check_root.cfg] action create (up to date)
            (up to date)

       Running handlers:
       Running handlers complete
       Chef Client finished, 12/77 resources updated in 14 seconds
       Finished converging <default-ubuntu-trusty64> (0m41.27s).
-----> Kitchen is finished. (0m41.63s)
~~~

### Finally the recipe.

This is how the recipe would look like.

~~~ yaml
# Here we are creating the group called `sysadmin`
# User and Password details will come from the `data_bags`,
#   check `data_bags` directory for more details.
default['users_setup']['groups'] = { 'sysadmin' => 2300 }

# Creating basic users for the setup.
#
# Creating a admin user/group for clouderamanager
# https://github.com/chef-cookbooks/users
# https://supermarket.chef.io/cookbooks/users#knife

node['users_setup']['groups'].each do | group_name,  group_id |
  users_manage group_name do
    group_id group_id
    action [:create]
    data_bag 'user_data_bag'
  end
end


# Update root password
users_manage 'root' do
    data_bag 'root_data_bag'
end
~~~
