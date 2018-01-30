---
title: Creating /etc/hosts file in Chef.
category: ['Linux', 'Centos', 'Redhat', 'Chef', 'Hadoop', 'Cluster']
tags: ['linux', 'centos', 'redhat', 'chef', 'hadoop', 'cluster']
---

We had a cluster environment which we needed to update the `/etc/hosts` file. Which would help communicate between the server over a private network. Our servers have multiple interfaces and we need them to communicate between each other using the private network. 

Goal for us is to create a `/etc/hosts` with all the nodes within the cluster with their private IP addresses.

### Chef Setup (Assumptions).

- We have multiple cluster.
- Each cluster has an environment set.
- We have a multiple interfaces on each node (But this solutions should work for single interface as well). 

### Steps we do to get information from each node.

1. We take 3 attributes to look for in the cluster.
	1. Each server with the cluster have specific `fqdn`.
	2. Private IP (string search).
	3. Interface to look for on the node. 
2. `all_nodes_in_cluster` will get all the nodes with that `fqdn` (This can be change based on requirement to `node`, `tags`, `roles`, `hostname`).
3. For every node we look for the specific interface and get the private IP.

### Before we start.

Before we start we need to check for our search criteria using  `knife search` more details [here](https://docs.chef.io/knife_search.html)

Below if an example to look for server `fqdn` containing string `env-lab-1`.

``` ruby 
knife search node "fqdn:*env-lab-1*"
```

## About the Recipe.


The interface information on hash is located as below 

``` ruby
node_in_cluster['network']['interfaces']['bond.007']['addresses']
```

This has a `Dictionary` with multiple values, we are specifically looking for `IP`.

```ruby
if private_interface[0].include? node['private_ip_search_filter']
```
Above we are looking for the interface which matches out search filter. Required information is in `private_interface[0]`

Here is how we would write the information in our `/etc/hosts` file, `IP`,`FQDN`,`HOSTNAME`.

``` ruby
puts "#{private_interface[0]} #{node_in_cluster['fqdn']} #{node_in_cluster['hostname']}"
```

Here is complete `ruby` code which does the same thing as in the `erb` template file.


``` ruby
all_nodes_in_cluster.each do |node_in_cluster|
  node_in_cluster['network']['interfaces'][int_to_look]['addresses'].each do |private_interface|
    if private_interface[0].include? node['private_ip_search_filter']
      puts "#{private_interface[0]} #{node_in_cluster['fqdn']} #{node_in_cluster['hostname']}"
    end
  end
end
```


### Attribute File.

``` ruby
# Example :  We take the search Criteria to generate /etc/hosts
default['env_search_filter'] = "fqdn:*lab-env-1*"
default['private_ip_search_filter'] = "192.168.149"
default['interface_to_look'] = 'bond.007'
```

### Recipe

``` ruby
# Search Criteria
all_nodes_in_cluster = search(:node, node['env_search_filter'])
int_to_look = node['interface_to_look']

template '/etc/hosts' do
  source 'etc_hosts_file.erb'
  mode '0755'
  owner 'root'
  group 'root'
  variables({
    all_nodes_in_cluster: all_nodes_in_cluster,
    int_to_look: int_to_look,
    private_ip_search_filter: node['private_ip_search_filter']
  })
end

```

### Template File `etc_hosts_file.erb`. 

``` ruby
127.0.0.1       localhost localhost.localdomain localhost4 localhost4.localdomain4
<% @all_nodes_in_cluster.each do |node_in_cluster| -%>
  <% node_in_cluster['network']['interfaces'][@int_to_look]['addresses'].each do |private_interface| -%>
    <% if private_interface[0].include? @private_ip_search_filter -%>
<%= private_interface[0] %>     <%= node_in_cluster['fqdn'] %>      <%= node_in_cluster['hostname'] %>  # Serial Number: <%= node_in_cluster['dmi']['system']['serial_number'] %> ( <%= node_in_cluster['dmi']['system']['manufacturer'] %> ) <%= node_in_cluster['dmi']['system']['product_name'] %>
    <% end -%>
  <% end -%>
<% end -%>
::1     localhost localhost.localdomain localhost6 localhost6.localdomain6

```

#### Disclaimer.

1. This does not look like an optimized solution, but something which worked for me. 
2. `search` method which will be run every 30mins, will query to get all information for all the nodes, which I think would be time/bandwidth consuming operation if we have a very large cluster. (A single node information was about 10000 lines of ruby `Hash` for our nodes).
3. If any one has a better way to do it, please post it in the comments below. Thanks :)