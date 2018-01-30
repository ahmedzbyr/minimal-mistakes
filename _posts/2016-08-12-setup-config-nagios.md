---
title: Setup/Configuration Nagios XI on Centos6.6
category: ['Centos', 'Nagios', 'Nagiosxi', 'Monitoring']
tags: ['centos', 'rhel', 'nagios', 'nagiosxi', 'monitoring']
---

Nagios monitors your entire IT infrastructure to ensure systems, applications, services, and business processes are functioning properly. In the event of a failure, Nagios can alert technical staff of the problem, allowing them to begin remediation processes before outages affect business processes, end-users, or customers.

Intro from : [https://www.nagios.org/about/](https://www.nagios.org/about/)

## Installation `Nagios XI` 

Here is the link to [Nagios XI 5.2.9](https://assets.nagios.com/downloads/nagiosxi/5/xi-5.2.9.tar.gz

Installation for Nagios for most of the cases is quite staight forward.

    tar xzf xi*.tar.gz
    cd nagiosxi
    ./fullinstall

And the script will do all the hardward and we are done.
Once install is complete, visit the URL displayed to complete the install   

**Running Full Install**

![Running full install](https://zubayr.github.io/images/nagios_0.0.0_install_complete.PNG)
   

**Installation Complete**

![Installation Complete](https://zubayr.github.io/images/nagios_0.0.1_install_complete.PNG)

**Initial Configuration**

![Initial configuration done](https://zubayr.github.io/images/nagios_0.0_install_complete.PNG)


## Configuration

### Checking if NRDS is working fine.

Go to the URL `http://nagios-server/nrdp` on the nagios server. [NRDP - Nagios Remote Data processor.]

![submit a query](https://zubayr.github.io/images/nagios_0.1_nrds_testing.PNG)

This is the response from the server.

![Response](https://zubayr.github.io/images/nagios_0.2_nrds_testing.PNG)

These submitted parameters will show up in the `Unconfigured Objects` page, which can be added directly.

![Unconfigured Objects](https://zubayr.github.io/images/nagios_0.3_nrds_testing.PNG)



### Creating an inbound configuration.

For servers to send-in their monitoring data, we need to create tokens so that the servers can authenticate themself.

#### Creating inbound config.

**NRDP Configuration**

![Inbound NRDP Configuration](https://zubayr.github.io/images/nagios_1_nrds_setting_tokens.PNG)

**NSCA Configuration** (Optional - Only if you are using this - Not recommended to use this as this is very old and not very reliable)

![Inbound NSCA Configuration](https://zubayr.github.io/images/nagios_2_nsca_setting_password.PNG)

### Creating NRDP Configuration for Windows and Linux.

**Creating Config for Linux**

![Config for Linux](https://zubayr.github.io/images/nagios_3.1_config_linux.PNG)

**Selecting Token and add config name**

![setting up info](https://zubayr.github.io/images/nagios_3.2_linux.PNG)

**Configuration Complete**

![Config Complete](https://zubayr.github.io/images/nagios_4_nrds_config_complete.PNG)

Now we are ready to setup the Clients.

Once the client starts sending results, if the host/service has not been configured yet it will be found in `Unconfigured Objects` and can easily be added to the monitoring config.


## Monitoring using NRDS on Linux.



![Config Complete](https://zubayr.github.io/images/nagios_4_nrds_config_complete.PNG)

The following commands can be run as root on all clients that will use the new_linux_config config.
The install process will perform the following operations:

* Install NRDS client
* Add a nagios user and group
* Add cron job to process checks
* Download plugins from the NRDP server

There are 2 items you need to modify below, HOSTNAME and INTERVAL.

* HOSTNAME - The name the client will send to the Nagios server as the host.
* INTERVAL - The frequency in minutes that you want the checks to be run. (1-59)

Below are the commands.

    cd /tmp

Get the tar file from the server.

    wget -O new_linux_config.tar.gz "http://<nagios_server_url>/nrdp/?cmd=nrdsgetclient&token=hardtoguess&configname=new_linux_config"

Unzip and install.

    gunzip -c new_linux_config.tar.gz | tar xf -
    cd clients
    ./installnrds [HOSTNAME] [INTERVAL_IN_MINUTES]

Command
    
    ./installnrds linux-server 5
    
This install the client and setup a `crontab` to send the parameters to the server.    
When the client `linux-server` sends the data you will see the details on the `Unconfigured Objects` page.

Select the parameters and add it to the `nagios` monitoring.
It would look similar to the page below. 

![nrds_testing](https://zubayr.github.io/images/nagios_0.3_nrds_testing.PNG)

## Monitoring using NRDS on windows.

**Download the executable from the link in the image**

![Config Complete](https://zubayr.github.io/images/nagios_4_nrds_config_complete.PNG)

**Agreement**

![Agreement](https://zubayr.github.io/images/nagios_6.1nrds_clinet_install_windows.PNG)

**Configure Server Name and Interval to send the data**

![Config](https://zubayr.github.io/images/nagios_6.2.nrds_clinet_install_windows.PNG)

**Create ini file and and setup scheduler task**

![ini file](https://zubayr.github.io/images/nagios_6.3.nrds_clinet_install_windows.PNG)

**Install**

![Install](https://zubayr.github.io/images/nagios_6.4.nrds_clinet_install_windows.PNG)

Once the client starts sending results, if the host/service has not been configured yet it will be found in `Unconfigured Objects` and can easily be added to the monitoring config. Example below.

![testing](https://zubayr.github.io/images/nagios_0.3_nrds_testing.PNG)

Select the parameters and add it to the `nagios` monitoring.
It would look similar to the page below. 

## Creating Groups for user to monitor specific servers.

Below are the steps to create a user groups.

1. Create a `contact` file in `/usr/local/nagios/etc/static`
2. Create a `contactgroup` file in `/usr/local/nagios/etc/static`
3. Create a host file which binds the contact group. 
4. And restart `nagios` service.

What are we trying to do.

1. On the `nagios` server we have 2 server which we monitor, A linux server and a windows server.
2. We want to make sure that the windows server is monitored by the windows admin team.
3. We will create a group and a user which can only see window server on the monitoring console.
4. This will help create set of groups, who can monitor only their servers which they are responsible for.  

Location of all the files on the server.

	┌─[zahmed][localhost][~]
	└─▪ cd /usr/local/nagios/etc/
	┌─[zahmed][localhost][/usr/local/nagios/etc]
	└─▪ ls
	cgi.cfg               hosttemplates.cfg  servicedependencies.cfg
	commands.cfg          import             serviceescalations.cfg
	contactgroups.cfg     nagios.cfg         serviceextinfo.cfg
	contacts.cfg          ndo2db.cfg         servicegroups.cfg
	contacttemplates.cfg  ndomod.cfg         services
	hostdependencies.cfg  nrpe.cfg           servicetemplates.cfg
	hostescalations.cfg   nsca.cfg           static
	hostextinfo.cfg       pnp                timeperiods.cfg
	hostgroups.cfg        resource.cfg
	hosts                 send_nsca.cfg
	┌─[zahmed][localhost][/usr/local/nagios/etc]
	└─▪


Creating a contact with `windowsnagiosadmin` user.


	┌─[zahmed][localhost][/usr/local/nagios/etc/static]
	└─▪ cat windows-server-contact.cfg
	define contact {
	        contact_name                            windowsnagiosadmin
	        alias                                   Windows Nagios Administrator
	        host_notification_period                nagiosadmin_notification_times
	        service_notification_period             nagiosadmin_notification_times
	        host_notification_options               d,u,r,f,s
	        service_notification_options            w,u,c,r,f,s
	        host_notification_commands              xi_host_notification_handler
	        service_notification_commands           xi_service_notification_handler
	        email                                   ahmed@gmail.com
	        #use                                     xi_contact_generic
	}

Create a `contactgroup` and add the above `windowsnagiosadmin` user as a member.

	┌─[zahmed][localhost][/usr/local/nagios/etc/static]
	└─▪ cat windows-server-contactgroup.cfg
	define contactgroup {
	    contactgroup_name                   windowsadmins
	    alias                               Windows Nagios Admin Group
	    members                             windowsnagiosadmin
	}

Now finally create a host configuration which is tied to the `windowsadmins` contact group above. 

	┌─[zahmed][localhost][/usr/local/nagios/etc/static]
	└─▪ cat nagios-test-windows.cfg
	define host {
	        host_name                       nagios-test-wındows
	        use                             xiwizard_passive_host
	        address                         nagios-test-wındows
	        max_check_attempts              5
	        check_interval                  5
	        retry_interval                  1
	        check_period                    xi_timeperiod_24x7
	        contact_groups                 windowsadmins
	        notification_interval           60
	        notification_period             xi_timeperiod_24x7
	        stalking_options                n
	        icon_image                      passiveobject.png
	        statusmap_image                 passiveobject.png
	        _xiwizard                       passiveobject
	        register                        1
	        }
	┌─[zahmed][localhost][/usr/local/nagios/etc/static]
	└─▪

Files in `static` directory.

	┌─[zahmed][localhost][/usr/local/nagios/etc/static]
	└─▪ ls -l
	total 24
	-rwxrwxr-x 1 apache nagios  661 Aug 12 11:27 windows-server-contact.cfg
	-rwxrwxr-x 1 apache nagios  211 Aug 12 11:57 windows-server-contactgroup.cfg
	-rwxrwxr-x 1 apache nagios  802 Aug 12 11:39 nagios-test-windows.cfg
	-rwxrwxr-x 1 apache nagios  878 Jun 14 20:43 xiobjects.cfg
	-rwxrwxr-x 1 apache nagios 4002 Jun 14 20:43 xitemplates.cfg
	-rwxrwxr-x 1 apache nagios    0 Jun 14 20:43 xitest.cfg
	-rwxr-xr-x 1 root   nagios  649 Aug 12 11:59 zahmedcontact.cfg

Check Nagios config.

	┌─[zahmed][localhost][/usr/local/nagios/etc/static]
	└─▪ sudo service nagios checkconfig
	[sudo] password for zahmed:
	Running configuration check...
	 OK.

All OK, then restart `nagios`

	┌─[zahmed][localhost][/usr/local/nagios/etc/static]
	└─▪ sudo service nagios restart
	Running configuration check...
	Stopping nagios:. done.
	Starting nagios: done.
	

Finally create a user on nagios which matches our configuration `windowsnagiosadmin`.

**Creating a `windowsnagiosadmin` user.**

![windows_user](https://zubayr.github.io/images/nagios_7.0.windows_user.PNG)

**Host when we login as `nagiosadmin`**

![login](https://zubayr.github.io/images/nagios_7.0.nagios_admin.PNG)

**Hosts when we login as `windowsnagiosadmin`**

![loggedin](https://zubayr.github.io/images/nagios_7.1.windows_user_login.PNG)


## Installation and Configuring `NSClient++` for Windows.

Download and install `NSClient++` on the windows server. [https://nsclient.org/download/](https://nsclient.org/download/)

**First Installing `NSClient++`**

![install](https://zubayr.github.io/images/nagios_8.1.nsclient_windows.PNG)

**Configure NSClient**

Update the `allowed_hosts` and `password`. And make sure to use `check_nt`

![config](https://zubayr.github.io/images/nagios_8.2.nsclient_windows.PNG)

**Install Complete**

![complete](https://zubayr.github.io/images/nagios_8.3.nsclient_windows.PNG)

**Check service is running**

![service check](https://zubayr.github.io/images/nagios_8.4.nsclient_windows.PNG)

### Configuring on the `nagios` server.

We need to create couple of configurations for the first time a windows machine is setup. After that all we have to do is add the `hosts` configuration. Everything else will be the same. (Unless we change the password on few of the windows machine then we have to update / create new command configuration)

1. Create a `host` configuration.
2. Add `services` configuration which will define what we want to monitor.
3. Create a `command` configuration. 

First lets create a file `new-windows-test.cfg` in `/usr/local/nagios/etc/static` directory.

We will be adding the `host` and `service` configuration into the same file. 

NOTE : `check_command` in the `service` definitions should be same as the `command_name` in `commands` configuration. Example: `check_nt_test` 

	┌─[zahmed][localhost][/usr/local/nagios/etc/static]
	└─▪ cat new-windows-test.cfg
	define host{
		use		            windows-server	; Inherit default values from a Windows server template (make sure you keep this line!)
		host_name	        winservertest
		alias		        My Windows Server
		address		        172.2.2.35
	    max_check_attempts  10
	    check_interval      5
	    retry_interval      1
	    check_period        24x7
	    contacts            nagiosadmin
	    notification_interval   30
	    notification_period     24x7
	    notification_options    d,r
	    register                1
		}
	    
	define service{
		use		            generic-service
		host_name		    winserver
		service_description	NSClient++ Version
		check_command		check_nt_test!CLIENTVERSION
		}
	    
	define service{
		use			        generic-service
		host_name			winserver
		service_description	Uptime
		check_command		check_nt_test!UPTIME
		}
	
	define service{
		use			        generic-service
		host_name			winserver
		service_description	CPU Load
		check_command		check_nt_test!CPULOAD!-l 5,80,90
		}
	
	define service{
		use			        generic-service
		host_name			winserver
		service_description	Memory Usage
		check_command		check_nt_test!MEMUSE!-w 80 -c 90
		}
	
	define service{
		use			        generic-service
		host_name			winserver
		service_description	C:\ Drive Space
		check_command		check_nt_test!USEDDISKSPACE!-l c -w 80 -c 90
		}
	    
	define service{
		use			        generic-service
		host_name			winserver
		service_description	W3SVC
		check_command		check_nt_test!SERVICESTATE!-d SHOWALL -l W3SVC
		}
	
	define service{
		use			        generic-service
		host_name			winserver
		service_description	Explorer
		check_command		check_nt_test!PROCSTATE!-d SHOWALL -l Explorer.exe
		}

    
First lets create a file `new-windows-test-command.cfg`, update the password `-s` below with the password we created on the windows machine. [Check image above]    
    
	┌─[zahmed][localhost][/usr/local/nagios/etc/static]
	└─▪ cat new-windows-test-command.cfg
	define command{
	        command_name    check_nt_test
	        command_line    $USER1$/check_nt -H $HOSTADDRESS$ -p 12489 -s nagios123 -v $ARG1$ $ARG2$
	        }


After we have configured new files.

	┌─[zahmed][localhost][/usr/local/nagios/etc/static]
	└─▪ ls -l new*
	-rw-r--r-- 1 apache nagios 1301 Aug 15 11:07 new-windows-test.cfg
	-rw-r--r-- 1 apache nagios  134 Aug 15 10:57 new-windows-test-command.cfg



Check if our configuration is fine.

	┌─[zahmed][localhost][/usr/local/nagios/etc/static]
	└─▪ sudo service nagios checkconfig
	[sudo] password for zahmed:
	Running configuration check...
	 OK.

Great!!, we have not errors, now we can restart `nagios`. 

	┌─[zahmed][localhost][/usr/local/nagios/etc]
	└─▪ sudo service nagios restart
	Running configuration check...
	Stopping nagios:. done.
	Starting nagios: done.

Restart complete and we are done.

### Checking the new host on the dashboard.

**New host `winservertest` is added to the dashboard.**

![service check](https://zubayr.github.io/images/nagios_9.1.nsclient_windows_dashboard.PNG)

**We are receiving monitoring data as well.**

![service check](https://zubayr.github.io/images/nagios_9.2.nsclient_windows_dashboard.PNG)

**Performance data.**

![service check](https://zubayr.github.io/images/nagios_9.3.nsclient_windows_dashboard.PNG)  

