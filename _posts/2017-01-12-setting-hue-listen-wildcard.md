---
title: Setting Hue to Listen on `0.0.0.0` [Cloudera]
category: ['Linux', 'Cloudera', 'Hadoop', 'Hue']
tags: ['linux', 'cloudera', 'hadoop']
---

We were working on setting up a cluster, but the Hue URL was set to a private IP of the server. As we had setup all the nodes to access each other using a private IP. But we wanted Hue to bind to public interface so that it can be accessed within the network. 

Bind Hue to wild card address.

1. Goto -> Hue Configuration -> search for `Bind Hue`.
2. Check `Bind Hue to Wildcard Address `
3. Restart Hue Server.  

We are done.