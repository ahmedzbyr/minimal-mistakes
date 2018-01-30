---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: Setting up ssl https On Nagios XI Server
category: ['Linux', 'Centos', 'Nagios', 'Nagiosxi', 'Ssl', 'Https', 'Monitoring']
tags: ['centos', 'linux', 'nagios', 'nagiosxi', 'ssl', 'https', 'monitoring']
---

HTTPS is a protocol for secure communication over a computer network which is widely used on the Internet. HTTPS consists of communication over Hypertext Transfer Protocol (HTTP) within a connection encrypted by Transport Layer Security or its predecessor, Secure Sockets Layer. The main motivation for HTTPS is authentication of the visited website and protection of the privacy and integrity of the exchanged data. [Intro Courtesy Wikipedia](https://en.wikipedia.org/wiki/HTTPS)

Full SSL support requires Nagios XI version 2011R1.6 or later.

### Before we start.

Check if the below packages are install, they should be if you are using latest Nagios XI, but check them anyways.

	yum install mod_ssl openssl
{: .language-ruby}

### Creating Key and Certificate

Lets generate the key for the server.

	openssl genrsa -out ca.key 2048
{: .language-ruby}

Output for the command.

	[ahmed@nagiosserver ~]$ openssl genrsa -out ca.key 2048
	Generating RSA private key, 2048 bit long modulus
	.................................................................................................................+++
	.....................................+++
	e is 65537 (0x10001)
{: .language-ruby}

Now we create the certificate.

	openssl req -new -key ca.key -out ca.csr
{: .language-ruby}

Here is the output for the command.

	[ahmed@nagiosserver ~]$ openssl req -new -key ca.key -out ca.csr
	You are about to be asked to enter information that will be incorporated
	into your certificate request.
	What you are about to enter is what is called a Distinguished Name or a DN.
	There are quite a few fields but you can leave some blank
	For some fields there will be a default value,
	If you enter '.', the field will be left blank.
	-----
	Country Name (2 letter code) [XX]:TR
	State or Province Name (full name) []:Istanbul
	Locality Name (eg, city) [Default City]:Istanbul
	Organization Name (eg, company) [Default Company Ltd]:Ahmed, Inc
	Organizational Unit Name (eg, section) []:
	Common Name (eg, your name or your server's hostname) []:nagiosserver.ahmed.com
	Email Address []:

	Please enter the following 'extra' attributes
	to be sent with your certificate request
	A challenge password []:
	An optional company name []:
{: .language-ruby}

We have not entered anything in the `extra` attributes, but this is fine.

Checking the certificate.

	openssl x509 -req -days 365 -in ca.csr -signkey ca.key -out ca.crt
{: .language-ruby}

Output.

	[ahmed@nagiosserver ~]$ openssl x509 -req -days 365 -in ca.csr -signkey ca.key -out ca.crt
	Signature ok
	subject=/C=TR/ST=Istanbul/L=Istanbul/O=Ahmed, Inc/CN=nagiosserver.ahmed.com
	Getting Private key
	[ahmed@nagiosserver ~]$
{: .language-ruby}

### Copy Key/Certificate to Specific Location.

Now we need to copy the certificate files to the correct location and set permissions:

	cp ca.crt /etc/pki/tls/certs
	cp ca.key ca.csr /etc/pki/tls/private/
{: .language-ruby}

Setting permissions.

	chmod go-rwx /etc/pki/tls/certs/ca.crt
	chmod go-rwx /etc/pki/tls/private/ca.key
{: .language-ruby}

### Update Apache Configuration

Open the `/etc/httpd/conf.d/ssl.conf`, find the following lines and update path, this is similar to what we copied earlier.

	SSLCertificateFile /etc/pki/tls/certs/ca.crt
	SSLCertificateKeyFile /etc/pki/tls/private/ca.key
{: .language-ruby}

Here is how the Configuration looks like.

![ssl cert](https://zubayr.github.io/images/nagios_ssl_1.png)

In that same file add the below contents just before `</VirtualHost>` tag:

	<IfModule mod_rewrite.c>
	RewriteEngine On
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteRule nagiosxi/api/v1/(.*)$ /usr/local/nagiosxi/html/api/v1/index.php?request=$1 [QSA,NC,L]
	</IfModule>
{: .language-ruby}

Here is how a part of the config looks like.

![IfModule](https://zubayr.github.io/images/nagios_ssl_2.png)

### Update `httpd.conf` Configuration.

Update `/etc/httpd/conf/httpd.conf`, Add the following lines to the end of the file:

	RewriteEngine On
	RewriteCond %{HTTPS} off
	RewriteRule (.*) https://%{HTTP_HOST}%{REQUEST_URI}
{: .language-ruby}

Here how the file looks like.

![httpd config](https://zubayr.github.io/images/nagios_ssl_3.png)

### Next we restart `httpd`

	sudo service httpd restart
{: .language-ruby}

Ouput.

	[ahmed@nagiosserver ~]$ sudo service httpd restart
	Stopping httpd:                                            [  OK  ]
	Starting httpd: httpd: apr_sockaddr_info_get() failed for nagiosserver
	httpd: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1 for ServerName [  OK  ]
{: .language-ruby}

Now we can go to `https://nagiosserver.ahmed.com/`, you get a warning about self certified certificate, add it to exception and we are ready.

### [Important] Now we update Nagios XI Configuration.

* First update the `config.inc.php` file.

Here is the path to the file.

	[ahmed@nagiosserver ~]# vim /usr/local/nagiosxi/html/config.inc.php
{: .language-ruby}

Update the below configuration in the file. (Currently `$cfg['use_https'] = false;`)

	// force http/https
	$cfg['use_https'] = true; // determines whether cron jobs and other scripts will force the use of HTTPS instead of HTTP
{: .language-ruby}


* Next logon to Nagios XI server as `nagiosadmin`.
* Go to `Admin` -> on the left pane `System Config` -> `System Settings` -> `General`. 
* Change the URL to `https`. Change `http://172.2.2.23/nagiosxi/` to `https://172.2.2.23/nagiosxi/`
* Next go to `Configure` on the top tab -> `Core Config Manager` -> On the left pane `Config Manager Admin` -> `Core Manager Settings` -> Change `Server Protocol` to  `HTTPS`

Restart `nagios`, `httpd`.

NOTE : If you are using filewall make sure to add the entry to `iptables`

	iptables -I INPUT -p tcp --dport 443 -j ACCEPT
	service iptables save
{: .language-ruby}

Now logon to the server. `https://nagiosserver.ahmed.com/`

