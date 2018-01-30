---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: Moving RRD file from 32bit to 64bit Architecture
category: ['Linux', 'Rhel', 'Ubuntu', 'Rrd', 'Nagios']
tags: ['ubuntu', 'centos', 'linux', 'rrd', 'nagios']
---

When we were working on a `nagios` monitoring system we were migrating from a 32bit nagios to a 64bit Architecture.
Most of the graphs are not working as the RRD was from an older 32bit architecture.

Location of perfdata on nagios server.

~~~ ruby 
[root@nagios-server perfdata]# pwd
/usr/local/nagios/share/perfdata
~~~ 

Error when we load the graph.

~~~ ruby 
ERROR: This RRD was created on another architecture
~~~ 

This can re solved by converting the exsisting 32bit RRD to XML and then restoring into the new 64bit Architecture.

### Creating a `dump` of the `rrd` file.

~~~ ruby
rrdtool dump stats.rrd > stats.xml
~~~ 

Move the XML file to the new server (64bit)

### Restore the XML file back.

~~~ ruby
rrdtool restore -f stats.xml stats.rrd
~~~ 

Testing if the RRD file is create fine, use below command.

~~~ ruby
rrdtool info stats.rrd
~~~ 

Now you should be able to see all the graphs on the server.   
