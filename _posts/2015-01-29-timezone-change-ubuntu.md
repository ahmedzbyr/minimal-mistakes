---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: Changing Timezone in Ubuntu server.
category: ['Linux', 'Ubuntu']
tags: ['linux', 'ubuntu', 'timezone']
---

Changing Timezone in Ubuntu server.

### Step 1: Check

	ahmed@server:~# date
	Thu Jan 29 02:38:55 EST 2015
	ahmed@server:~# more /etc/timezone
	US/Eastern

### Step 2: Change Timezone setting 

	ahmed@server:~# dpkg-reconfigure tzdata
	
	Current default time zone: 'Asia/Kolkata'
	Local time is now:      Thu Jan 29 13:09:54 IST 2015.
	Universal Time is now:  Thu Jan 29 07:39:54 UTC 2015.
	
	ahmed@server:~# date
	Thu Jan 29 13:09:57 IST 2015
	ahmed@server:~#

### Step 3: Also make sure to restart cron service as this will still have the old time.

	ahmed@server:~# service crond restart
	ahmed@server:~# service nginx restart

And we are done.