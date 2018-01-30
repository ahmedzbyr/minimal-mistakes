---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: Chef Workstation Setup on Windows Machine.
category: ['Windows', 'Chef', 'Chefdk']
tags: ['windows', 'chef', 'chefdk']
---

The Chef Development Kit (ChefDK) brings the best-of-breed development tools built by the awesome Chef community to your workstation with just a few clicks. Download your package and start coding Chef in seconds.

## Install Instructions

Once the package has been downloaded, double-click the .msi file to run the MSI installer. Use all of the default installation options.
The chef command and the other commands included with the Chef Development Kit should now be available for use.

###  Download and configure.

![Download File](http://zubayr.github.io/images/chef_work_download.png)

![Download File](http://zubayr.github.io/images/chef_work_download2.png)


### Install Vagrant.

[https://www.vagrantup.com/downloads.html](https://www.vagrantup.com/downloads.html)


### Install VirtualBox.

[https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads)


## Setting up the Environment.

### Creating a Blank `cookbook` using `chefdk`

Command.

	chef generate cookbook my_first_cookbook

Here is the output.

	PS C:\Users\zubair.ahmed\centos_vagrant> chef generate cookbook my_first_cookbook
	Installing Cookbook Gems:
	Compiling Cookbooks...
	Recipe: code_generator::cookbook
	  * directory[C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook] action create
	    - create new directory C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook
	  * template[C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/metadata.rb] action create_if_missing
	    - create new file C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/metadata.rb
	    - update content in file C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/metadata.rb from none to a4d2ff
	    (diff output suppressed by config)

        * template[C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/README.md] action create_if_missing
	    - create new file C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/README.md
	    - update content in file C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/README.md from none to b84cc5

	    (diff output suppressed by config)
	  * cookbook_file[C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/chefignore] action create
	    - create new file C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/chefignore
	    - update content in file C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/chefignore from none to 15fac5
	    (diff output suppressed by config)
	  * cookbook_file[C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/Berksfile] action create_if_missing
	    - create new file C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/Berksfile
	    - update content in file C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/Berksfile from none to 9f08dc

	    (diff output suppressed by config)

      * template[C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/.kitchen.yml] action create_if_missing
	    - create new file C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/.kitchen.yml
	    - update content in file C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/.kitchen.yml from none to d53214 
        (diff output suppressed by config)
        
	  * directory[C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/test/integration/default/serverspec] action	create
	    - create new directory C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/test/integration/default/serverspec
	  * directory[C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/test/integration/helpers/serverspec] action create
	    - create new directory C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/test/integration/helpers/serverspec
	  * cookbook_file[C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/test/integration/helpers/serverspec/spec_helper.rb] action create_if_missing
	    - create new file C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/test/integration/helpers/serverspec/spec_helper.rb
	    - update content in file C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/test/integration/helpers/serverspec/spec_helper.rb from none to d85df4
	    (diff output suppressed by config)
        
	  * template[C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/test/integration/default/serverspec/default_spec.rb] action create_if_missing
	    - create new file C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/test/integration/default/serverspec/default_spec.rb
	    - update content in file C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/test/integration/default/serverspec/default_spec.rb from none to 3c5e72
	    (diff output suppressed by config)
        
	  * directory[C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/spec/unit/recipes] action create
	    - create new directory C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/spec/unit/recipes
	  * cookbook_file[C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/spec/spec_helper.rb] action create_if_missing
	    - create new file C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/spec/spec_helper.rb
	    - update content in file C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/spec/spec_helper.rb from none to 587075
	    (diff output suppressed by config)
        
	  * template[C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/spec/unit/recipes/default_spec.rb] action create_if_missing
	    - create new file C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/spec/unit/recipes/default_spec.rb
	    - update content in file C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/spec/unit/recipes/default_spec.rb from none to ed26b2
	    (diff output suppressed by config)
        
	  * directory[C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/recipes] action create
	    - create new directory C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/recipes
	  * template[C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/recipes/default.rb] action create_if_missing
	    - create new file C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/recipes/default.rb
	    - update content in file C:/Users/zubair.ahmed/centos_vagrant/my_first_cookbook/recipes/default.rb from none to d2c7b9
	    (diff output suppressed by config)


### Update `.kitchen.yml` file as below.

We are just updating the platforms from centos 7 to 6.5.
You can keep it as it is if you are looking for centos 7.

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
	  - name: centos-6.5

	suites:
	  - name: default
	    run_list:
	      - recipe[my_first_cookbook::default]
	    attributes:


### Creating the VM.

Checking the list of VM which we have in the `kitchen.yml` file.

	kitchen list

Output.

	PS C:\Users\zubair.ahmed\centos_vagrant\my_first_cookbook> kitchen.bat list
	Instance           Driver   Provisioner  Verifier  Transport  Last Action
	default-centos-65  Vagrant  ChefZero     Busser    Ssh        <Not Created>

Creating a VM.

	kitchen create default-centos-65

Output.

	PS C:\Users\zubair.ahmed\centos_vagrant\my_first_cookbook> kitchen.bat create default-centos-65
	-----> Starting Kitchen (v1.7.3)
	-----> Creating <default-centos-65>...
	       Bringing machine 'default' up with 'virtualbox' provider...
	       ==> default: Importing base box 'bento/centos-6.5'...
	==> default: Matching MAC address for NAT networking...
	       ==> default: Setting the name of the VM: kitchen-my_first_cookbook-default-centos-65_default_1465204398970_64998
	       ==> default: Clearing any previously set network interfaces...
	       ==> default: Preparing network interfaces based on configuration...
	           default: Adapter 1: nat
	       ==> default: Forwarding ports...
	           default: 22 (guest) => 2222 (host) (adapter 1)
	       ==> default: Booting VM...
	       ==> default: Waiting for machine to boot. This may take a few minutes...
	           default: SSH address: 127.0.0.1:2222
	           default: SSH username: vagrant
	           default: SSH auth method: private key
	           default: Warning: Remote connection disconnect. Retrying...
	           default:
	           default: Vagrant insecure key detected. Vagrant will automatically replace
	           default: this with a newly generated keypair for better security.
	           default:
	           default: Inserting generated public key within guest...
	           default: Removing insecure key from the guest if it's present...
	           default: Key inserted! Disconnecting and reconnecting using new SSH key...
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
	       Vagrant instance <default-centos-65> created.
	       Finished creating <default-centos-65> (1m58.85s).
	-----> Kitchen is finished. (2m7.25s)

![Download File](http://zubayr.github.io/images/chef_work_create.png)
    
### Make some basic changes to the `my_first_cookbook`.

Update `default.rb` in `recipe` as below. File path : `${HOME}/my_first_cookbook/recipes/default.rb`

    package 'java-1.7.0-openjdk' do
      action :install
    end

    #
    # Creating `group`
    group 'tomcat' do
      action :create # This is a default behaviour
    end

    # Creating user `tomcat`
    user 'tomcat' do
      group 'tomcat'
      home '/opt/tomcat'
      shell '/bin/nologin'
      uid '1234'
      action :create
    end


    # Create directory

    directory ('/opt/tomcat') do
      owner 'tomcat'
      group 'tomcat'
    end


    #
    # Downloading Apache tomcat.
    #
    remote_file ('/tmp/apache-tomcat-8.0.33.tar.gz') do
      source 'http://mirror.cc.columbia.edu/pub/software/apache/tomcat/tomcat-8/v8.0.33/bin/apache-tomcat-8.0.33.tar.gz'
    end
    #
    # # TODO: Change this.
    #
    execute 'tar xvzf /tmp/apache-tomcat-8.0.33.tar.gz -C /opt/tomcat/ --strip-components=1' do
      not_if do ::File.exists?('/opt/tomcat/conf/server.xml') end
    end

    # TODO: Need to update this ...
    execute 'chown -R tomcat:tomcat /opt/tomcat'

    directory '/opt/tomcat/conf' do
      mode 0070
    end

    execute '/opt/tomcat/bin/startup.sh'

Update the test spec. File Path : `${HOME}/my_first_cookbook/test/integration/default/serverspec/default_spec.rb`

    require 'spec_helper'

    describe 'tomcat::default' do

      describe command('curl --noproxy 127.0.0.1 http://127.0.0.1:8080') do
        its(:stdout) { should match /tomcat/ }
      end


      describe file ('/opt/tomcat') do
        it { should exist }
        it { should be_directory }
      end

      describe group ('tomcat') do
        it { should exist }
      end

      describe user ('tomcat') do
        it { should exist }
        it { should belong_to_group 'tomcat' }
        it { should have_home_directory '/opt/tomcat' }
        it { should have_login_shell '/bin/nologin' }
      end

      describe file ('/opt/tomcat/conf') do
        it { should exist }
        it { should be_mode 70 }
      end


    end



### Converge the cookbook with VM.

This will converge the recipe with the VM.

	kitchen.bat converge default-centos-65

Output.

    PS C:\Users\zubair.ahmed\centos_vagrant\tomcat> kitchen.bat converge default-centos-65
    -----> Starting Kitchen (v1.7.3)
    -----> Converging <default-centos-65>...
           Preparing files for transfer
           Preparing dna.json
           Resolving cookbook dependencies with Berkshelf 4.3.2...
           Removing non-cookbook files before transfer
           Preparing solo.rb
    -----> Chef Omnibus installation detected (install only if missing)
           Transferring files to <default-centos-65>
           Starting Chef Client, version 12.11.18
           resolving cookbooks for run list: ["tomcat::default"]
           Synchronizing Cookbooks:
             - tomcat (0.1.0)
           Installing Cookbook Gems:
           Compiling Cookbooks...
           Converging 9 resources
           Recipe: tomcat::default
             * yum_package[java-1.7.0-openjdk] action install
               - install version 1.7.0.101-2.6.6.4.el6_8 of package java-1.7.0-openjdk
             * group[tomcat] action create
               - create group tomcat
             * user[tomcat] action create
               - create user tomcat
             * directory[/opt/tomcat] action create (up to date)
             * remote_file[/tmp/apache-tomcat-8.0.33.tar.gz] action create
               - create new file /tmp/apache-tomcat-8.0.33.tar.gz
               - update content in file /tmp/apache-tomcat-8.0.33.tar.gz from none to c77873
               (new content is binary, diff output suppressed)
               - restore selinux security context
             * execute[tar xvzf /tmp/apache-tomcat-8.0.33.tar.gz -C /opt/tomcat/ --strip-components=1] action run
               - execute tar xvzf /tmp/apache-tomcat-8.0.33.tar.gz -C /opt/tomcat/ --strip-components=1
             * execute[chown -R tomcat:tomcat /opt/tomcat] action run
               - execute chown -R tomcat:tomcat /opt/tomcat
             * directory[/opt/tomcat/conf] action create
               - change mode from '0755' to '070'
               - restore selinux security context
             * execute[/opt/tomcat/bin/startup.sh] action run
               - execute /opt/tomcat/bin/startup.sh

           Running handlers:
           Running handlers complete
           Chef Client finished, 8/9 resources updated in 53 seconds
           Finished converging <default-centos-65> (0m56.68s).
    -----> Kitchen is finished. (1m2.05s)


![Download File](http://zubayr.github.io/images/chef_work_converge.png)

### Verify the converge.

Verify the converge and check if all the file and services are installed.

	kitchen.bat verify default-centos-65

Output.

    PS C:\Users\zubair.ahmed\centos_vagrant\tomcat> kitchen.bat verify default-centos-65
    -----> Starting Kitchen (v1.7.3)
    -----> Setting up <default-centos-65>...
           Finished setting up <default-centos-65> (0m0.00s).
    -----> Verifying <default-centos-65>...
           Preparing files for transfer
    -----> Installing Busser (busser)
    Fetching: thor-0.19.0.gem (100%)
           Successfully installed thor-0.19.0
    Fetching: busser-0.7.1.gem (100%)
           Successfully installed busser-0.7.1
           2 gems installed
           Installing Busser plugins: busser-serverspec
           Plugin serverspec installed (version 0.5.9)
    -----> Running postinstall for serverspec plugin
           Suite path directory /tmp/verifier/suites does not exist, skipping.
           Transferring files to <default-centos-65>
    -----> Running serverspec test suite
    -----> Installing Serverspec..
    Fetching: sfl-2.2.gem (100%)
    Fetching: net-telnet-0.1.1.gem (100%)
    Fetching: net-ssh-3.1.1.gem (100%)
    Fetching: net-scp-1.2.1.gem (100%)
    Fetching: specinfra-2.59.0.gem (100%)
    Fetching: rspec-support-3.4.1.gem (100%)
    Fetching: diff-lcs-1.2.5.gem (100%)
    Fetching: rspec-expectations-3.4.0.gem (100%)
    Fetching: rspec-core-3.4.4.gem (100%)
    Fetching: rspec-its-1.2.0.gem (100%)
    Fetching: rspec-mocks-3.4.1.gem (100%)
    Fetching: rspec-3.4.0.gem (100%)
    Fetching: multi_json-1.12.1.gem (100%)
    Fetching: serverspec-2.36.0.gem (100%)
    -----> serverspec installed (version 2.36.0)
           /opt/chef/embedded/bin/ruby -I/tmp/verifier/suites/serverspec -I/tmp/verifier/gems/gems/rspec-support-3.4.1/lib:/
    tmp/verifier/gems/gems/rspec-core-3.4.4/lib /opt/chef/embedded/bin/rspec --pattern /tmp/verifier/suites/serverspec/\*\*/
    \*_spec.rb --color --format documentation --default-path /tmp/verifier/suites/serverspec

           tomcat::default
             Command "curl --noproxy 127.0.0.1 http://127.0.0.1:8080"
               stdout
                 should match /tomcat/
             File "/opt/tomcat"
               should exist
               should be directory
             Group "tomcat"
               should exist
             User "tomcat"
               should exist
               should belong to group "tomcat"
               should have home directory "/opt/tomcat"
               should have login shell "/bin/nologin"
             File "/opt/tomcat/conf"
               should exist
               should be mode 70

           Finished in 1.92 seconds (files took 0.38732 seconds to load)
           10 examples, 0 failures

           Finished verifying <default-centos-65> (0m26.23s).
    -----> Kitchen is finished. (0m31.66s)

Important part in the above output is as below. All the services are fine. We have `10 test examples` and `0 failures`.

           tomcat::default
             Command "curl --noproxy 127.0.0.1 http://127.0.0.1:8080"
               stdout
                 should match /tomcat/
             File "/opt/tomcat"
               should exist
               should be directory
             Group "tomcat"
               should exist
             User "tomcat"
               should exist
               should belong to group "tomcat"
               should have home directory "/opt/tomcat"
               should have login shell "/bin/nologin"
             File "/opt/tomcat/conf"
               should exist
               should be mode 70

           Finished in 1.92 seconds (files took 0.38732 seconds to load)
           10 examples, 0 failures

           Finished verifying <default-centos-65> (0m26.23s).
    -----> Kitchen is finished. (0m31.66s)


![Download File](http://zubayr.github.io/images/chef_work_verify.png)

Your first cookbook is ready.