---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: knife Quick Reference
category: ['Centos', 'Nagios', 'Monitoring']
tags: ['centos', 'rhel', 'nagios', 'opensource', 'monitoring']
---

This is a quick reference for few of the commands which I use often.

## Inital setup

~~~ ruby
knife ssl fetch
knife node list
knife bootstrap 172.2.2.34 --node-name nagiosxi-test-linux --ssh-port 22 --ssh-user root --ssh-password nagiosxi@123 --sudo
knife bootstrap windows winrm 172.22.2.222 --winrm-user 'Administrator' --winrm-password 'Nagios2234' --node-name nagiosxi_test_windows_client --winrm-ssl-verify-mode verify_none -V -y --run-list 'recipe[chef-client]'
~~~

## Creating data bags

~~~ ruby
knife data bag create starter-databag cmadmin
knife data bag edit starter-databag cmadmin
~~~

## Auto generate cookbook

~~~ ruby
chef generate cookbook <cookbook_name>
chef generate cookbook <cookbook_name> --copyright "Zubair AHMED" --email "zubayr.a@gmail.com" --license "mit"
chef generate attribute <attribute_file_name> --copyright "Zubair AHMED" --email "zubayr.a@gmail.com" --license "mit"
chef generate recipe <recipe_name> --copyright "Zubair AHMED" --email "zubayr.a@gmail.com" --license "mit"
~~~

## Upload and Download cookbooks

~~~ ruby
knife cookbook upload <cookbook_name>
knife cookbook upload --all
knife cookbook site install <cookbook_name>
knife cookbook site search apache*
knife cookbook site show haproxy
~~~

## Testing

~~~ ruby
kitchen create
kitchen converge
kitchen verify
kitchen destroy
~~~



## Here is the complete reference.

Courtesy [https://github.com/chef/quick-reference](https://github.com/chef/quick-reference)

![Quick Reference](https://raw.githubusercontent.com/chef/quick-reference/master/qr_knife_web.png)
