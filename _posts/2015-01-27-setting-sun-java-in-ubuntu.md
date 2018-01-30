---
title: Setting SUN Java for Ubuntu.
category: ['Linux']
tags: ['linux', 'install-java', 'java', 'ubuntu', 'jdk']
---

The `Java Development Kit` (JDK) is an implementation of either one of the Java SE, Java EE or Java ME platforms released by Oracle Corporation in the form of a binary product aimed at Java developers on Solaris, Linux, Mac OS X or Windows. This is currently not avaiable on `apt-get` repo, so here is quick way to setup `java` on ubuntu server/desktop with `tar` ball. 

### First create directory for Java

	ahmed@ahmed-server:~/sun-java#  mkdir -p /usr/lib/jvm/
	ahmed@ahmed-server:~/sun-java#  tar xvzf jdk1.7.0_75.tgz -C /usr/lib/jvm/

### Setting Alternatives

	ahmed@ahmed-server:~/sun-java#  sudo update-alternatives \
                       --install "/usr/bin/java" "java" "/usr/lib/jvm/jdk1.7.0_75/bin/java" 1
	ahmed@ahmed-server:~/sun-java#  sudo update-alternatives \
                       --install "/usr/bin/javac" "javac" "/usr/lib/jvm/jdk1.7.0_75/bin/javac" 1
	ahmed@ahmed-server:~/sun-java#  sudo update-alternatives \
                       --install "/usr/bin/javaws" "javaws" "/usr/lib/jvm/jdk1.7.0_75/bin/javaws" 1
                            
	update-alternatives: using /usr/lib/jvm/jdk1.7.0_75/bin/javaws to 
                                                    provide /usr/bin/javaws (javaws) in auto mode.

### Make sure we have the right permission.	
	
	ahmed@ahmed-server:~/sun-java#  sudo chmod a+x /usr/bin/java
	ahmed@ahmed-server:~/sun-java#  sudo chmod a+x /usr/bin/javac
	ahmed@ahmed-server:~/sun-java#  sudo chmod a+x /usr/bin/javaws
	ahmed@ahmed-server:/usr/lib/jvm#  sudo chown -R root:root /usr/lib/jvm/jdk1.7.0_75

### Configuration of Alternatives
	
	ahmed@ahmed-server:/usr/lib/jvm#  sudo update-alternatives --config java
	There are 3 choices for the alternative java (providing /usr/bin/java).

	  Selection    Path                                            Priority   Status
	------------------------------------------------------------
	* 0            /usr/lib/jvm/java-6-openjdk-amd64/jre/bin/java   1061      auto mode
	  1            /usr/lib/jvm/java-6-openjdk-amd64/jre/bin/java   1061      manual mode
	  2            /usr/lib/jvm/java-7-openjdk-amd64/jre/bin/java   1051      manual mode
	  3            /usr/lib/jvm/jdk1.7.0_75/bin/java                1         manual mode

	Press enter to keep the current choice[*], or type selection number: 3
	update-alternatives: using /usr/lib/jvm/jdk1.7.0_75/bin/java to provide 
                                                                /usr/bin/java (java) in manual mode.

	ahmed@ahmed-server:/usr/lib/jvm#  sudo update-alternatives --config javac
	There are 2 choices for the alternative javac (providing /usr/bin/javac).

	  Selection    Path                                         Priority   Status
	------------------------------------------------------------
	* 0            /usr/lib/jvm/java-7-openjdk-amd64/bin/javac   1051      auto mode
	  1            /usr/lib/jvm/java-7-openjdk-amd64/bin/javac   1051      manual mode
	  2            /usr/lib/jvm/jdk1.7.0_75/bin/javac            1         manual mode

	Press enter to keep the current choice[*], or type selection number: 2
	update-alternatives: using /usr/lib/jvm/jdk1.7.0_75/bin/javac to provide 
                                                            /usr/bin/javac (javac) in manual mode.
	ahmed@ahmed-server:/usr/lib/jvm#  sudo update-alternatives --config javawc
	update-alternatives: error: no alternatives for javawc.

### Checking version

	ahmed@ahmed-server:/usr/lib/jvm#  java -version
	java version "1.7.0_75"
	Java(TM) SE Runtime Environment (build 1.7.0_75-b13)
	Java HotSpot(TM) 64-Bit Server VM (build 24.75-b04, mixed mode)
	ahmed@ahmed-server:/usr/lib/jvm#  javac -version
	javac 1.7.0_75
	ahmed@ahmed-server:/usr/lib/jvm# 