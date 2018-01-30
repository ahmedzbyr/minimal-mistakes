---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: Installing SNMP Builder using `zabbix-extra` on Zabbix Version 2.4.
category: ['Linux', 'Zabbix', 'Snmp', 'Trapper', 'Nagios', 'Monitoring']
tags: ['zabbix', 'snmp', 'trap', 'centos', 'linux', 'nagios', 'monitoring']
---

SNMP Builder/Extra is an add-in for Zabbix. It provides new web interface components to browse MIB trees and values. SNMP OIDs can then be automatically converted into Zabbix items and inserted into a template. The underlying snmpbuilder script uses calls to NetSNMP in order to communicate with devices on the network.

Maintainers: `Zabbix forum members`
Original Author: `giapnguyen`

Features

* **MIB Browser**: you can select default MIB files or your own device MIBs to get an OID tree. Click on the tree to retrieve value and information about a OID. Click to transform the OID to a Zabbix item.
* **MIB Browser**: uses snmpv2c to connect to remote devices using netSNMP module. To use snmpv1, modify the scripted SNMP calls to use v1 instead of v2c. v1/v2 change is not exposed in the php interface.
* **OID Table support**: it assume that a OID whose name's end with string 'Table' is a OID Table. OID Table will retrieve with all it's indexes. Click on the cell to select the index as Zabbix's item. If my above assumption is wrong, you can use the checkbox .view as table. to switch between table and normal view.
* **Column selection**: On OID Table, click on a header will select a whole column as Zabbix's items. It's useful if you create SNMP template for a 48 ports switch 8-).
Auto convert: This is implemented as a simple conversion from SNMP to Zabbix item.


For installation in a new Zabbix web interface will require only two steps:

1. Download the Zabbix-ons installation script;
2. Installation of Zabbix 2.1-ons;

Got this from  `http://www.zabbix.com/third_party_tools.php` very helpful extra.

More on this extra in the link below.

	https://github.com/zubayr/zabbix-extras
	
More setup information can be found here, but you might need to use `google translate`.

	http://spinola.net.br/blog/?p=544
	
	
###  Download the Zabbix Extra from the below link.

![files](http://zubayr.github.io/images/zabbix_extra_files.JPG)

	[root@localhost Downloads]# wget https://github.com/zubayr/zabbix-extras/archive/ZE2.1.zip
	[root@localhost Downloads]# unzip ZE2.1.zip
	[root@localhost Downloads]# cd zabbix-extras-ZE2.1
	[root@localhost zabbix-extras-ZE2.1]# pwd
	/home/ahmed/Downloads/zabbix-extras-ZE2.1
	[root@localhost zabbix-extras-ZE2.1]# ls -l
	total 256
	drwxrwxrwx 4 ahmed ahmed  4096 Dec  9 12:09 include
	-rwxrw-rw- 1 ahmed ahmed 34904 Jan 13  2015 instalaExtras.sh
	drwxrwxrwx 3 ahmed ahmed  4096 Dec  9 12:09 jqplot
	-rwxrw-rw- 1 ahmed ahmed  2241 Jan 13  2015 README.md
	-rwxrw-rw- 1 ahmed ahmed  1458 Jan 13  2015 zbxe-arvore.php
	-rwxrw-rw- 1 ahmed ahmed  4234 Jan 13  2015 zbxe-cat-chart-builder.php
	-rwxrw-rw- 1 ahmed ahmed 18438 Jan 13  2015 zbxe-cat.php
	-rwxrw-rw- 1 ahmed ahmed 37946 Jan 13  2015 zbxe-em.php
	-rwxrw-rw- 1 ahmed ahmed  1495 Jan 13  2015 zbxe-geolocation.php
	-rwxrw-rw- 1 ahmed ahmed 54959 Jan 13  2015 zbxe-inicia-bd.php
	-rwxrw-rw- 1 ahmed ahmed  4524 Jan 13  2015 zbxe_item_test.php
	-rwxrw-rw- 1 ahmed ahmed  1471 Jan 13  2015 zbxe-logo.php
	-rwxrw-rw- 1 ahmed ahmed  7944 Jan 13  2015 zbxe-ns.php
	-rwxrw-rw- 1 ahmed ahmed 14348 Jan 13  2015 zbxe-sc.php
	-rwxrw-rw- 1 ahmed ahmed 36114 Jan 13  2015 zbxe-snmp-builder.php
	-rwxrw-rw- 1 ahmed ahmed  2030 Jan 13  2015 zbxe-translation-export.php

	
###  Now lets execute the script `instalaExtras.sh`. 	
	
	[root@localhost zabbix-extras-ZE2.1]# sh instalaExtras.sh
	

####  Selecting Language. 	

![Selecting Language.](http://zubayr.github.io/images/zabbix_extra_lang.JPG)
	
	instalaExtras.sh: line 784: unalias: rm: not found
	-->Mensagem Versao do Linux - OK (centos - 6.6)


							lZabbix Extras Installer [2.1.3]qqk
							x Informe o idioma (Enter the     x
							x language for the installer)     x
							x lqqqqqqqqqqqqqqqqqqqqqqqqqqqqqk x
							x x ( ) pt  Portugues / Brasil  x x
							x x (*) en  English             x x
							x mqqqqqqqqqqqqqqqqqqqqqqqqqqqqqj x
							x                                 x
							x                                 x
							tqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqu
							x     <  OK  >   <Cancel>         x
							mqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqj


####  Enter the path to zabbix web installation, usually it is `/usr/share/zabbix/`.

![Web Path.](http://zubayr.github.io/images/zabbix_extra_zabbix_web_path.JPG)

							lqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqk
							x This installer will add an extra menu to the   x
							x end of the menu bar of your environment. For   x
							x installation are needed to inform some         x
							x parameters.                                    x
							x Please enter the path to the zabbix frontend   x
							x lqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqk x
							x x/usr/share/zabbix/                          x x
							x mqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqj x
							tqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqu
							x           <  OK  >      <Cancel>               x
							mqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqj



####  URL to zabbix `http://192.168.126.129/zabbix`, change it as per requirement.

![Web URL.](http://zubayr.github.io/images/zabbix_extra_zabbix_web_url.JPG)

							lqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqk
							x This installer will add an extra menu to the end of   x
							x the menu bar of your environment. For installation    x
							x are needed to inform some parameters.                 x
							x Please enter the URL to the zabbix frontend (using    x
							x localhost)                                            x
							x lqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqk x
							x xhttp://192.168.126.129/zabbix                      x x
							x mqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqj x
							tqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqu
							x               <  OK  >        <Cancel>                x
							mqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqj


	
####  Enter path for `php.ini`, by default it is in `/etc/php.ini`.

![php.ini path.](http://zubayr.github.io/images/zabbix_extra_php_ini_path.JPG)


							lqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqk
							x This installer will configure the PHP:        x
							x short_open_tag, activating it. This step is   x
							x required to install and ZabTree ZabGeo.       x
							x Please enter the path to php.ini.             x
							x lqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqk x
							x x/etc/php.ini                               x x
							x mqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqj x
							tqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqu
							x           <  OK  >      <Cancel>              x
							mqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqj



####  Select downloading package from internet. Or we have to download all the packages from internet and save it in `/tmp`.

	[root@zbx-master-server ~]# ls -l /tmp/plugin*
	-rw-r--r-- 1 root root    7747 Dec  9 14:35 /tmp/pluginArvoreDaemon.zip
	-rw-r--r-- 1 root root   14367 Dec  9 14:35 /tmp/pluginArvoreJS.zip
	-rw-r--r-- 1 root root  144877 Dec  9 14:34 /tmp/pluginArvore.zip
	-rw-r--r-- 1 root root     193 Dec  9 14:36 /tmp/pluginExtrasBD.htm
	-rw-r--r-- 1 root root 1648347 Dec  9 14:36 /tmp/pluginExtras.zip
	-rw-r--r-- 1 root root  125767 Dec  9 14:34 /tmp/pluginGeo.zip
	-rw-r--r-- 1 root root  529784 Dec  9 14:35 /tmp/pluginSNMPB.zip
	[root@zbx-master-server ~]#

Here are location for the downloads of there is no interenet connection the zabbix server.

		[root@zbx-master-server ~]# wget \
		https://github.com/aristotelesaraujo/zabbix-geolocation/archive/master.zip \
		-O /tmp/pluginGeo.zip
		
		[root@zbx-master-server ~]# wget \
		https://github.com/SpawW/zabbix-service-tree/archive/2.4.zip -O /tmp/pluginArvore.zip
		
		[root@zbx-master-server ~]# wget \
		https://github.com/SpawW/zabbix-service-tree-daemon/archive/master.zip \
		-O /tmp/pluginArvoreDaemon.zip
		
		[root@zbx-master-server ~]# wget \
		https://github.com/SpawW/html5-tree-graph/archive/master.zip -O /tmp/pluginArvoreJS.zip
		

####  Currently we will select `internet`.

![pkg from internet.](http://zubayr.github.io/images/zabbix_extra_download_pkg_from_internet.JPG)


							lqqqqqqqqqqqqqqqqqqqqqqqqqqDownload Filesqqqqqqqqqqqqqqqqqqqqqqqqqqqqk
							x Download the patch files (S) (S = Yes)?                            x
							x lqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqk x
							x x (*) S  Get from internet latest version of patchs (recomended) x x
							x x ( ) N  Use files saved in /tmp                                 x x
							x mqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqj x
							x                                                                    x
							x                                                                    x
							tqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqu
							x                   <  OK  >          <Cancel>                       x
							mqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqj


####  If we have installed this package earlier then we will get the below menu. **[OPTIONAL]**.

![older version.](http://zubayr.github.io/images/zabbix_extra_older_version_preserve.JPG)

							lqqqZabbix Extras BD Update [2.1.3]qqqqqqk
							x A previous installation was detected.  x
							x Do you want to REPLACE the data from   x
							x the tables by the new ZBXE data? If    x
							x the installation is damaged you must   x
							x choose this option!                    x
							x lqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqk x
							x x    ( ) S  Re-create zbxe tables    x x
							x x    (*) N  Preserve zbxe tables     x x
							x mqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqj x
							x                                        x
							x                                        x
							tqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqu
							x         <  OK  >     <Cancel>          x
							mqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqj

####  Downloading and setting up configuration. 


	-->Mensagem  Recriar banco [N]
	-->Mensagem Configurando suporte a customizacoes nos mapas...
	-->Mensagem  Upgrade install (cores mapa)...
	-->Mensagem  Upgrade install (empresa)...
	-->Mensagem  Upgrade install (funcao adicional para cores)...
	-->Mensagem  Upgrade install (customizando cores padroes)...
	-->Mensagem  Upgrade install (esconde titulo)...
	-->Mensagem Configurando suporte a logotipo personalizado...
	-->Mensagem  Upgrade install (logotipo)...
	-->Mensagem  Upgrade install (Adicionando suporte para verificacao em tempo real de chave)...
	-->Mensagem  Upgrade install (Adicionando botao para suporte a verificacao em tempo real de chave)...
	-->Mensagem Configurando portlet com link para itens nao suportados...
	-->Mensagem  Upgrade install (NS)...
	--2015-12-09 14:34:22--  https://github.com/aristotelesaraujo/zabbix-geolocation/archive/master.zip
	Resolving github.com... 192.30.252.131
	Connecting to github.com|192.30.252.131|:443... connected.
	HTTP request sent, awaiting response... 302 Found
	Location: https://codeload.github.com/aristotelesaraujo/zabbix-geolocation/zip/master [following]
	--2015-12-09 14:34:28--  https://codeload.github.com/aristotelesaraujo/zabbix-geolocation/zip/master
	Resolving codeload.github.com... 192.30.252.146
	Connecting to codeload.github.com|192.30.252.146|:443... connected.
	HTTP request sent, awaiting response... 200 OK
	Length: 125767 (123K) [application/zip]
	Saving to: “/tmp/pluginGeo.zip”

	100%[==========================================================>] 125,767     38.5K/s   in 3.2s

	2015-12-09 14:34:37 (38.5 KB/s) - “/tmp/pluginGeo.zip” saved [125767/125767]

	--2015-12-09 14:34:37--  https://github.com/SpawW/zabbix-service-tree/archive/2.4.zip
	Resolving github.com... 192.30.252.131
	Connecting to github.com|192.30.252.131|:443... connected.
	HTTP request sent, awaiting response... 302 Found
	Location: https://codeload.github.com/SpawW/zabbix-service-tree/zip/2.4 [following]
	--2015-12-09 14:34:42--  https://codeload.github.com/SpawW/zabbix-service-tree/zip/2.4
	Resolving codeload.github.com... 192.30.252.146
	Connecting to codeload.github.com|192.30.252.146|:443... connected.
	HTTP request sent, awaiting response... 200 OK
	Length: 144877 (141K) [application/zip]
	Saving to: “/tmp/pluginArvore.zip”

	100%[==========================================================>] 144,877     14.6K/s   in 9.7s

	2015-12-09 14:34:58 (14.6 KB/s) - “/tmp/pluginArvore.zip” saved [144877/144877]

	--2015-12-09 14:34:58--  https://github.com/SpawW/zabbix-service-tree-daemon/archive/master.zip
	Resolving github.com... 192.30.252.131
	Connecting to github.com|192.30.252.131|:443... connected.
	HTTP request sent, awaiting response... 302 Found
	Location: https://codeload.github.com/SpawW/zabbix-service-tree-daemon/zip/master [following]
	--2015-12-09 14:35:03--  https://codeload.github.com/SpawW/zabbix-service-tree-daemon/zip/master
	Resolving codeload.github.com... 192.30.252.146
	Connecting to codeload.github.com|192.30.252.146|:443... connected.
	HTTP request sent, awaiting response... 200 OK
	Length: 7747 (7.6K) [application/zip]
	Saving to: “/tmp/pluginArvoreDaemon.zip”

	100%[======================================================>] 7,747       21.3K/s   in 0.4s

	2015-12-09 14:35:09 (21.3 KB/s) - “/tmp/pluginArvoreDaemon.zip” saved [7747/7747]

	--2015-12-09 14:35:09--  https://github.com/SpawW/html5-tree-graph/archive/master.zip
	Resolving github.com... 192.30.252.131
	Connecting to github.com|192.30.252.131|:443... connected.
	HTTP request sent, awaiting response... 302 Found
	Location: https://codeload.github.com/SpawW/html5-tree-graph/zip/master [following]
	--2015-12-09 14:35:14--  https://codeload.github.com/SpawW/html5-tree-graph/zip/master
	Resolving codeload.github.com... 192.30.252.146
	Connecting to codeload.github.com|192.30.252.146|:443... connected.
	HTTP request sent, awaiting response... 200 OK
	Length: 14367 (14K) [application/zip]
	Saving to: “/tmp/pluginArvoreJS.zip”

	100%[=================================================>] 14,367      10.6K/s   in 1.3s

	2015-12-09 14:35:21 (10.6 KB/s) - “/tmp/pluginArvoreJS.zip” saved [14367/14367]

	--2015-12-09 14:35:22--  https://github.com/SpawW/snmpbuilder/archive/master.zip
	Resolving github.com... 192.30.252.131
	Connecting to github.com|192.30.252.131|:443... connected.
	HTTP request sent, awaiting response... 302 Found
	Location: https://codeload.github.com/SpawW/snmpbuilder/zip/master [following]
	--2015-12-09 14:35:30--  https://codeload.github.com/SpawW/snmpbuilder/zip/master
	Resolving codeload.github.com... 192.30.252.146
	Connecting to codeload.github.com|192.30.252.146|:443... connected.
	HTTP request sent, awaiting response... 200 OK
	Length: 529784 (517K) [application/zip]
	Saving to: “/tmp/pluginSNMPB.zip”

	100%[=========================================================>] 529,784     55.1K/s   in 12s

	2015-12-09 14:35:48 (42.9 KB/s) - “/tmp/pluginSNMPB.zip” saved [529784/529784]

	-->Mensagem  Upgrade install (Adicionando scripts para o snmp-builder-1)...
	-->Mensagem  Upgrade install (Adicionando scripts para o snmp-builder-2)...
	-->Mensagem  Upgrade install (Adicionando scripts para o snmp-builder-2)...
	--2015-12-09 14:35:49--  https://github.com/SpawW/zabbix-extras/archive/ZE2.1.zip
	Resolving github.com... 192.30.252.131
	Connecting to github.com|192.30.252.131|:443... connected.
	HTTP request sent, awaiting response... 302 Found
	Location: https://codeload.github.com/SpawW/zabbix-extras/zip/ZE2.1 [following]
	--2015-12-09 14:35:54--  https://codeload.github.com/SpawW/zabbix-extras/zip/ZE2.1
	Resolving codeload.github.com... 192.30.252.146
	Connecting to codeload.github.com|192.30.252.146|:443... connected.
	HTTP request sent, awaiting response... 200 OK
	Length: 1648347 (1.6M) [application/zip]
	Saving to: “/tmp/pluginExtras.zip”

	100%[==========================================================>] 1,648,347   98.5K/s   in 25s

	2015-12-09 14:36:25 (65.2 KB/s) - “/tmp/pluginExtras.zip” saved [1648347/1648347]

	-->Mensagem Iniciando banco de dados...
	2015-12-09  http://192.168.126.129/zabbix/zbxe-inicia-bd.php?p_modo_install=N&p_versao_zbx=2.4.7
	Connecting to 192.168.126.129:80... connected.
	HTTP request sent, awaiting response... 200 OK
	Length: 193 [text/html]
	Saving to: “/tmp/pluginExtrasBD.htm”

	100%[===========================================================>] 193         --.-K/s   in 0s

	2015-12-09 14:36:27 (49.6 MB/s) - “/tmp/pluginExtrasBD.htm” saved [193/193]

	-->Mensagem Instalando patch de literais...
	-->Mensagem Existe instalacao previa no arquivo... removendo customizacao do patch literal!
	-->Mensagem Instalando tags identificadoras do menu...
	-->Mensagem Instalando menus customizados...
	-->Mensagem  Upgrade install (NS)...
	-->Mensagem  Upgrade install (personalizando profile)...
	-->Mensagem  Upgrade install (Ajustando fields para modo global)...
	-->Mensagem  Upgrade install (Adicionando parametros personalizados)...
	-->Mensagem  Upgrade install (profile.php Adicionando regra de negocio)...
	-->Mensagem  Upgrade install (personalizando profile)...
	-->Mensagem  Upgrade install (Ajustando fields para modo global)...
	-->Mensagem  Upgrade install (Adicionando parametros personalizados)...
	-->Mensagem  Clean install (users.php Adicionando regra de negocio)...
	-->Mensagem  Upgrade install (Adicionando aba do extras no profile)...
	-->Mensagem Parametros usados para instalacao:
	-->Mensagem URL do Zabbix: [http://192.168.126.129/zabbix]
	-->Mensagem Path do frontend Zabbix: [/usr/share/zabbix/]
	-->Mensagem Path do php.ini: [/etc/php.ini]
	-->Mensagem Se for necessario suporte favor enviar, por e-mail, os arquivos abaixo:
	-->Mensagem /tmp/pluginExtrasBD.htm
	-->Mensagem /tmp/upgZabbix/logInstall.log
	[root@zbx-master-server zabbix-extras-ZE2.1]#

Installation complete. 


###  Now we add all the required MIBs in the directory below.

	[root@zbx-master-server mibs]# pwd
	/usr/share/zabbix/extras/snmp-builder/mibs

Once you add the MIBs to ``, change the permission on `mibs` directory to `777`
	
	[root@zbx-master-server mibs]# cd /usr/share/zabbix/extras/snmp-builder/
	[root@zbx-master-server snmp-builder]# chmod 777 -R mibs

And we are done, Now we should see all the MIBs in the SNMP Builder UI.
	
	
###  Next we see the result on the web UI.

Here is the UI screenshot. Now we can see an extra tab on top.

![Web UI SNMP Builder](http://zubayr.github.io/images/zabbix_extra_snmp_builder.JPG)

DONE!! 

