---
title: Unable to locate package oracle-java7-installer - Ubuntu 13
category: ['Linux', 'Ubuntu']
tags: ['linux', 'ubuntu', 'java', 'jdk']
---

Was installing Java today, this is an easy install thanks to `ppa:webupd8team/java`, but when I tried it was not working, but has worked for me all this while. 

Little goggling around found me a solution. Here is full conversation if you wanted to have a look `http://ubuntuforums.org/showthread.php?t=2048793`

Installing JAVA on Ubuntu is simple. Do the below to add the repository and run the below command 'sudo apt-get install oracle-java7-installer' to install. 

##  Error.

	ahmed@ubuntu:~$ sudo add-apt-repository ppa:webupd8team/java
	ahmed@ubuntu:~$ sudo apt-get update
	ahmed@ubuntu:~$ sudo apt-get install oracle-java7-installer

	ahmed@ubuntu:~$ sudo apt-get install oracle-java7-installer
	Reading package lists... Done
	Building dependency tree       
	Reading state information... Done
	E: Unable to locate package oracle-java7-installer


## Solution.

Elevating to `root`.

	ahmed@ubuntu:~$ sudo su

Adding to `deb`.

	root@ubuntu:~# echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu precise main" \
                                                    > /etc/apt/sources.list.d/webupd8team-java.list

Adding to `deb-src`
	
	root@ubuntu:~# echo "deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu precise main" \
                                                    >> /etc/apt/sources.list.d/webupd8team-java.list
	
Making sure we have the key as well.

	root@ubuntu:~# apt-key adv --keyserver keyserver.ubuntu.com --recv-keys EEA14886

Lets `update` our `apt-get` repository.
	
	root@ubuntu:~# apt-get update
	
Now Install Oracle Java

	root@ubuntu:~# apt-get install oracle-java7-installer
	<..loads of verbose...>

	root@ubuntu:~# exit

Checking Version

	ahmed@ubuntu:~$ 
	ahmed@ubuntu:~$ java -version
	java version "1.7.0_60"
	Java(TM) SE Runtime Environment (build 1.7.0_60-b19)
	Java HotSpot(TM) 64-Bit Server VM (build 24.60-b09, mixed mode)
	ahmed@ubuntu:~$

All done. !!! 