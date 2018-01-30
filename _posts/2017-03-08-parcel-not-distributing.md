---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: Parcel Not Distributing Cloudera CDH.
category: ['Linux', 'Centos', 'Redhat', 'Cloudera', 'Hadoop', 'Cluster']
tags: ['linux', 'centos', 'redhat', 'cloudera', 'hadoop', 'cluster']
---

We were deploying one of the cluster on our lab environment which is used by everyone.
So the lab has it own share of stale information on it.

During installation we notice that the distribution is not working. There could be couple of reasons.
This was the second time we are having this issue.

1. Check `/etc/hosts` file if we have all the server names added correctly
2. Second reason would be due to the fact that, one of the installation was terminated midway, leaving a stale config which set the status to `ACTIVATING` for CDH parcel. So when we try to install, parcel was not distributing.  
3. Again there could be similar issue if we do not have enough space on the node for `/opt/cloudera`.


### Solution: 

- Deactivating parcel and retry.

``` python
curl -u username:password -X POST http://adminnode:7180/api/v14/clusters/<myCluster>/parcels/products/CDH/versions/5.10.0-1.cdh5.10.0.p0.41/commands/deactivate
``` 

- Check for space and increase space for `/opt/cloudera/`
- Most of the time should see the issue on the logs, server-logs: `/var/log/cloudera-scm-server/cloudera-scm-server.log` or agent-logs: `/var/log/cloudera-scm-agent/cloudera-scm-agent.log`
