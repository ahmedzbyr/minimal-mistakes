---
title: Package Installer for Cygwin [apt-cyg].
category: ['Linux', 'Windows', 'Cygwin']
tags: ['linux', 'windows', 'cygwin']
---

After a longtime I was on my windows machine and had to make it feel more like my linux machine. So install the thing what everyone else does `cygwin`.
Surpise my custom `.bashrc` and `.vimrc` worked without any issues. Good !! had the bashrc update vimrc update, we are back to linux .. like :)

My custom linux environment - [_howto_](https://zubayr.github.io/linux-env/).

Then I realized there is no way to install package from cygwin terminal.
Then I found below script `apt-cyg` which is really nice.

Package Installer - **apt-cyg**  [`http://github.com/transcode-open/apt-cyg`](http://github.com/transcode-open/apt-cyg)

### Installation

`apt-cyg` is a simple script, copy below script to home directory on `cygwin`

Here is the link [`https://github.com/transcode-open/apt-cyg/blob/master/apt-cyg`](https://github.com/transcode-open/apt-cyg/blob/master/apt-cyg)

Execute below command.

~~~ ruby
install apt-cyg /bin
~~~

Now we can use - Example use of `apt-cyg`

~~~ ruby
apt-cyg install nano
apt-cyg install lynx
~~~

Output

~~~ ruby
┌─[Zubair][AHMD-WRK-HORSE][~]
└─▪ apt-cyg install lynx
Installing lynx
--2016-09-28 12:49:39--  http://cygwin.mirror.constant.com//x86_64/release/lynx/lynx-2.8.7-2.tar.bz2
Resolving cygwin.mirror.constant.com (cygwin.mirror.constant.com)... 108.61.5.83
Connecting to cygwin.mirror.constant.com (cygwin.mirror.constant.com)|108.61.5.83|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1746879 (1.7M) [application/octet-stream]
Saving to: ‘lynx-2.8.7-2.tar.bz2’

lynx-2.8.7-2.tar.bz2           100%[==================================>]   1.67M   181KB/s    in 12s

2016-09-28 12:49:52 (146 KB/s) - ‘lynx-2.8.7-2.tar.bz2’ saved [1746879/1746879]

lynx-2.8.7-2.tar.bz2: OK
Unpacking...
Package lynx requires the following packages, installing:
bash cygwin libiconv2 libintl8 libncursesw10 libopenssl100 zlib0
Package bash is already installed, skipping
Package cygwin is already installed, skipping
Package libiconv2 is already installed, skipping
Package libintl8 is already installed, skipping
Package libncursesw10 is already installed, skipping
Package libopenssl100 is already installed, skipping
Package zlib0 is already installed, skipping
Running /etc/postinstall/lynx.sh
Package lynx installed
~~~

Now we are good. !!!
