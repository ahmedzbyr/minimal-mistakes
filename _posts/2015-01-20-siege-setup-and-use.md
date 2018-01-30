---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: Setup Seige on Centos 6.5, Kernel 2.6, CPU x86_64
category: ['Linux', 'Testing']
tags: ['load-testing', 'performance', 'seige', 'tsung', 'testing']
---

Siege is an http load testing and benchmarking utility. It was designed to let web developers measure their code under duress, to see how it will stand up to load on the internet. Siege supports basic authentication, cookies, HTTP, HTTPS and FTP protocols. It lets its user hit a server with a configurable number of simulated clients. Those clients place the server `under siege`.

## Installation Procedure.

First install prerequisites gcc/make on server. If Ubuntu

	sudo apt-get install gcc make
	
On Centos/Redhat

	sudo yum install gcc make


Step to Installation.

	$ ./configure
	$ make
	$ sudo make install

Files Installed.
	
	siege          -->    SIEGE_HOME/bin/siege
	bombardment    -->    SIEGE_HOME/bin/bombardment
	siege2csv      -->    SIEGE_HOME/bin/siege2csv
	.siegerc       -->    $HOME/.siegerc
	siege.1        -->    SIEGE_HOME/man/man1/siege.1
	bombardment.1  -->    SIEGE_HOME/man/man1/bombardment.1
	siege2csv.1    -->    SIEGE_HOME/man/man1/siege2csv.1
	layingsiege.1  -->    SIEGE_HOME/man/man1/layingsiege.1
	urls_text.1    -->    SIEGE_HOME/man/man1/urls_txt.1
	urls.txt       -->    SIEGE_HOME/etc/urls.txt 

Usage for `siege` command.

	root@SIDCLB:~# siege
	SIEGE 3.0.9
	Usage: siege [options]
	       siege [options] URL
	       siege -g URL
	Options:
	  -V, --version             VERSION, prints the version number.
	  -h, --help                HELP, prints this section.
	  -C, --config              CONFIGURATION, show the current config.
	  -v, --verbose             VERBOSE, prints notification to screen.
	  -q, --quiet               QUIET turns verbose off and suppresses output.
	  -g, --get                 GET, pull down HTTP headers and display the
	                            transaction. Great for application debugging.
	  -c, --concurrent=NUM      CONCURRENT users, default is 10
	  -i, --internet            INTERNET user simulation, hits URLs randomly.
	  -b, --benchmark           BENCHMARK: no delays between requests.
	  -t, --time=NUMm           TIMED testing where "m" is modifier S, M, or H
	                            ex: --time=1H, one hour test.
	  -r, --reps=NUM            REPS, number of times to run the test.
	  -f, --file=FILE           FILE, select a specific URLS FILE.
	  -R, --rc=FILE             RC, specify an siegerc file
	  -l, --log[=FILE]          LOG to FILE. If FILE is not specified, the
	                            default is used: PREFIX/var/siege.log
	  -m, --mark="text"         MARK, mark the log file with a string.
	  -d, --delay=NUM           Time DELAY, random delay before each requst
	                            between 1 and NUM. (NOT COUNTED IN STATS)
	  -H, --header="text"       Add a header to request (can be many)
	  -A, --user-agent="text"   Sets User-Agent in request
	  -T, --content-type="text" Sets Content-Type in request 

For more detailed information, consult the man pages:

	$ man siege
	$ man layingsiege
	$ man siege.config

All the siege man pages are also available online:

	http://www.joedog.org/siege/docs/man/index.html

OR, read the html manual, doc/manual.html  The manual is also available online:

	http://www.joedog.org/siege/docs/manual.html 


## Usage Procedure

### Basic Commands

To test the GET request, you would run

	siege http://example.com/ -c 100 -r 100

To test the POST request, you would run

	siege -H 'Content-Type:application/json' "http://example.com/ POST < ./data.json" -c 10 -r 1000

