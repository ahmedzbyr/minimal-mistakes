---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: Check Port on Remote Server CentOS 6.6/RHEL 6
category: ['Linux', 'Centos', 'Nc']
tags: ['centos', 'linux', 'nc']
---

Checking port available on a remote machine using nc command instead of telnet.
Same command can be used to check on a remote server as well, change the 127.0.0.1 with IP address of the server.

Single Port Check

    nc -zv 127.0.0.1 80

Range of ports:

    nc -zv 127.0.0.1 20-30

Output :

    root@server ~]# nc -zv 127.0.0.1 443
    Connection to 127.0.0.1 443 port [tcp/https] succeeded!
    [root@server ~]# nc -zv 127.0.0.1 443-445
    Connection to 127.0.0.1 443 port [tcp/https] succeeded!
    Connection to 127.0.0.1 444 port [tcp/snpp] succeeded!
    nc: connect to 127.0.0.1 port 445 (tcp) failed: Connection refused
    [root@server ~]#