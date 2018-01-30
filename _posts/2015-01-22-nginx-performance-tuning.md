---
title: Performance Tuning for `nginx`
category: ['Linux']
tags: ['linux', 'nginx', 'performance', 'tuning', 'load-balancing']
---

Nginx (pronounced `engine x`) is a web server with a strong focus on high concurrency, performance and low memory usage. It can also act as a reverse proxy server for `HTTP`, `HTTPS`, `SMTP`, `POP3`, and `IMAP` protocols, as well as a load balancer and an `HTTP cache`.

### Configuration Information. 

**worker_processes**: The number of NGINX worker processes. 
In most cases, running one worker process per CPU core works well. 
This can be achieved by setting this directive to “`auto`”. 
There are times when you may want to increase this number, such as when the work processes have to do a lot of disk I/O. The default is 1.

Also this controls number of worker processes Nginx is running.
Set `worker_processes` = number of processors in your system.
To find out how many processors you have on your server, run the following command:


	grep processor /proc/cpuinfo | wc -l


**Current Configuration**


	worker_processes auto;


**`worker_connections`**

If you are running very high traffic sites, its better to increase value of worker_connections. Default is 768.
Theoretically, nginx can handle max clients = worker_processes ** worker_connections

We use `worker_connections = 10240`

	worker_connections  10240;


**`worker_rlimit_nofile`**

Increase number of files opened byworker_process.
This directive is not present by default. You can add it in /etc/nginx/nginx.conf in main section (belowworker_processes)

We use `worker_rlimit_nofile 100000;`


	worker_rlimit_nofile 100000;


**`keepalive_requests`**: The number of requests a client can make over a single keepalive connection. 
The default is 100, but a much higher value can be especially useful for testing when the load generating tool is sending many requests from a single client.

Current we use `keepalive_requests 20480;`


	#  Use this value for testing once done comment it.
	keepalive_requests 20480;



Setting **`access_log`** with buffer.
Write the data to file once we have reached the buffer.
I/O Operations slow down processing. 


        access_log  /var/log/nginx/access.log  main buffer=10m;


More Information Below.

	http://www.slashroot.in/nginx-web-server-performance-tuning-how-to-do-it
	https://rtcamp.com/tutorials/nginx/optimization/
	http://nginx.com/blog/tuning-nginx/
	https://www.digitalocean.com/community/tutorials/how-to-optimize-nginx-configuration