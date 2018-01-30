---
title: Windows Testing Using Kitchen Chef
category: ['Windows', 'Ubuntu', 'Chef', 'Kitchen', 'Testing']
tags: ['windows', 'ubuntu', 'chef', 'kitchen', 'testing']
---

Kitchen-Vagrant has the capability to spin up a windows instance for testing.
To make it work you will need the `vagrant-winrm` to be installted on the workstation.

### Installing `vagrant-winrm`

``` ruby
┌─[ahmed][zubair-HP-ProBook][~/work]
└─▪ vagrant plugin install vagrant-winrm
```

Once you have have installed you might still get the below error.

``` ruby
┌─[ahmed][zubair-HP-ProBook][~/work]
└─▪ kitchen list
>>>>>> ------Exception-------
>>>>>> Class: Kitchen::UserError
>>>>>> Message: WinRM Transport requires the vagrant-winrm Vagrant plugin to properly communicate with this Vagrant VM. Please install this plugin with: `vagrant plugin install vagrant-winrm' and try again.
>>>>>>
>>>>>> Please see .kitchen/logs/kitchen.log for more details
>>>>>> Also try running `kitchen diagnose --all` for configuration
```

## Download Windows Box.

There is a nice repos which creates windows vagrant box.

```ruby
git clone https://github.com/boxcutter/windows.git
```

Here is the output.

```ruby
┌─[ahmed][zubair-HP-ProBook][~/work]
└─▪ git clone https://github.com/boxcutter/windows.git
Cloning into 'windows'...
remote: Counting objects: 2929, done.
remote: Total 2929 (delta 0), reused 0 (delta 0), pack-reused 2929
Receiving objects: 100% (2929/2929), 6.40 MiB | 1010.00 KiB/s, done.
Resolving deltas: 100% (2318/2318), done.
Checking connectivity... done.
```

#### Download and List of Available Boxes.

``` ruby
┌─[ahmed][zubair-HP-ProBook][~/work]
└─▪ cd windows/
┌─[ahmed][zubair-HP-ProBook][±][master ✓][~/work/windows]
└─▪ ls
AUTHORS                                win2008r2-web.json
bin                                    win2008r2-web-ssh.json
box                                    win2012-datacenter-cygwin.json
CHANGELOG.md                           win2012-datacenter.json
eval-win10x64-enterprise-cygwin.json   win2012-datacenter-ssh.json
eval-win10x64-enterprise.json          win2012r2-datacenter-cygwin.json
eval-win10x64-enterprise-ssh.json      win2012r2-datacenter.json
eval-win10x86-enterprise-cygwin.json   win2012r2-datacenter-ssh.json
eval-win10x86-enterprise.json          win2012r2-standardcore-cygwin.json
eval-win10x86-enterprise-ssh.json      win2012r2-standardcore.json
eval-win2008r2-datacenter-cygwin.json  win2012r2-standardcore-ssh.json
eval-win2008r2-datacenter.json         win2012r2-standard-cygwin.json
eval-win2008r2-datacenter-ssh.json     win2012r2-standard.json
eval-win2008r2-standard-cygwin.json    win2012r2-standard-ssh.json
eval-win2008r2-standard.json           win2012-standard-cygwin.json
eval-win2008r2-standard-ssh.json       win2012-standard.json
eval-win2012r2-datacenter-cygwin.json  win2012-standard-ssh.json
eval-win2012r2-datacenter.json         win7x64-enterprise-cygwin.json
eval-win2012r2-datacenter-ssh.json     win7x64-enterprise.json
eval-win2012r2-standard-cygwin.json    win7x64-enterprise-ssh.json
eval-win2012r2-standard.json           win7x64-pro-cygwin.json
eval-win2012r2-standard-ssh.json       win7x64-pro.json
eval-win7x64-enterprise-cygwin.json    win7x64-pro-ssh.json
eval-win7x64-enterprise.json           win7x86-enterprise-cygwin.json
eval-win7x64-enterprise-ssh.json       win7x86-enterprise.json
eval-win7x86-enterprise-cygwin.json    win7x86-enterprise-ssh.json
eval-win7x86-enterprise.json           win7x86-pro-cygwin.json
eval-win7x86-enterprise-ssh.json       win7x86-pro.json
eval-win81x64-enterprise-cygwin.json   win7x86-pro-ssh.json
eval-win81x64-enterprise.json          win81x64-enterprise-cygwin.json
eval-win81x64-enterprise-ssh.json      win81x64-enterprise.json
eval-win81x86-enterprise-cygwin.json   win81x64-enterprise-ssh.json
eval-win81x86-enterprise.json          win81x64-pro-cygwin.json
eval-win81x86-enterprise-ssh.json      win81x64-pro.json
eval-win8x64-enterprise-cygwin.json    win81x64-pro-ssh.json
eval-win8x64-enterprise.json           win81x86-enterprise-cygwin.json
eval-win8x64-enterprise-ssh.json       win81x86-enterprise.json
floppy                                 win81x86-enterprise-ssh.json
LICENSE                                win81x86-pro-cygwin.json
Makefile                               win81x86-pro.json
README.md                              win81x86-pro-ssh.json
script                                 win8x64-enterprise-cygwin.json
test                                   win8x64-enterprise.json
tpl                                    win8x64-enterprise-ssh.json
VERSION                                win8x64-pro-cygwin.json
win2008r2-datacenter-cygwin.json       win8x64-pro.json
win2008r2-datacenter.json              win8x64-pro-ssh.json
win2008r2-datacenter-ssh.json          win8x86-enterprise-cygwin.json
win2008r2-enterprise-cygwin.json       win8x86-enterprise.json
win2008r2-enterprise.json              win8x86-enterprise-ssh.json
win2008r2-enterprise-ssh.json          win8x86-pro-cygwin.json
win2008r2-standard-cygwin.json         win8x86-pro.json
win2008r2-standard.json                win8x86-pro-ssh.json
win2008r2-standard-ssh.json            wip
win2008r2-web-cygwin.json              wsim

```

#### We get error `packer` not found.

``` ruby
┌─[ahmed][zubair-HP-ProBook][±][master ✓][~/work/windows]
└─▪ make virtualbox/eval-win2012r2-standard
rm -rf output-virtualbox-iso
mkdir -p box/virtualbox
packer build -only=virtualbox-iso -var 'cm=nocm' -var 'version=1.0.4' -var 'update=false' -var 'headless=false' -var "shutdown_command=shutdown /s /t 10 /f /d p:4:1 /c Packer_Shutdown" -var "iso_url=http://download.microsoft.com/download/6/2/A/62A76ABB-9990-4EFC-A4FE-C7D698DAEB96/9600.16384.WINBLUE_RTM.130821-1623_X64FRE_SERVER_EVAL_EN-US-IRM_SSS_X64FREE_EN-US_DV5.ISO" -var "iso_checksum=7e3f89dbff163e259ca9b0d1f078daafd2fed513" eval-win2012r2-standard.json
/bin/sh: 1: packer: not found
Makefile:428: recipe for target 'box/virtualbox/eval-win2012r2-standard-nocm-1.0.4.box' failed
make: *** [box/virtualbox

```

#### Let us install `packer` from `hashicorp`

Location : [`https://releases.hashicorp.com/packer/0.10.1/packer_0.10.1_linux_amd64.zip`](https://releases.hashicorp.com/packer/0.10.1/packer_0.10.1_linux_amd64.zip)

``` ruby
┌─[ahmed][zubair-HP-ProBook][~/work]
└─▪ wget https://releases.hashicorp.com/packer/0.10.1/packer_0.10.1_linux_amd64.zip
--2016-09-22 11:21:14--  https://releases.hashicorp.com/packer/0.10.1/packer_0.10.1_linux_amd64.zip
Resolving releases.hashicorp.com (releases.hashicorp.com)... 151.101.12.69
Connecting to releases.hashicorp.com (releases.hashicorp.com)|151.101.12.69|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 8985735 (8.6M) [application/zip]
Saving to: ‘packer_0.10.1_linux_amd64.zip’

packer_0.10.1_linux_ 100%[======================>]   8.57M   204KB/s    in 29s

2016-09-22 11:21:44 (298 KB/s) - ‘packer_0.10.1_linux_amd64.zip’ saved [8985735/8985735]
```

#### Unzip and Install `packer`

Unpacking.

```ruby
┌─[ahmed][zubair-HP-ProBook][~/work]
└─▪ unzip packer_0.10.1_linux_amd64.zip
Archive:  packer_0.10.1_linux_amd64.zip
  inflating: packer
┌─[ahmed][zubair-HP-ProBook][~/work]
└─▪ ls
backups    configs          others  packer_0.10.1_linux_amd64.zip  tech_documents
chef-repo  hepsi-chef-repo  packer  scripts                        windows
```

Copy packer to `/usr/local/sbin/`

```ruby
┌─[ahmed][zubair-HP-ProBook][~/work]
└─▪ sudo cp packer /usr/local/sbin/
[sudo] password for ahmed:
```

Now we are ready to use `packer`

```ruby
┌─[ahmed][zubair-HP-ProBook][~/work]
└─▪ packer
usage: packer [--version] [--help] <command> [<args>]

Available commands are:
    build       build image(s) from template
    fix         fixes templates from old versions of packer
    inspect     see components of a template
    push        push a template and supporting files to a Packer build service
    validate    check that a template is valid
    version     Prints the Packer version

┌─[ahmed][zubair-HP-ProBook][~/work]
└─▪ packer --version
0.10.1
```

#### Now lets install `eval-win2012r2-standard`.


``` ruby

┌─[ahmed][zubair-HP-ProBook][±][master ✓][~/work/windows]
└─▪ make virtualbox/eval-win2012r2-standard
rm -rf output-virtualbox-iso
mkdir -p box/virtualbox
packer build -only=virtualbox-iso -var 'cm=nocm' -var 'version=1.0.4' -var 'update=false' -var 'headless=false' -var "shutdown_command=shutdown /s /t 10 /f /d p:4:1 /c Packer_Shutdown" -var "iso_url=http://download.microsoft.com/download/6/2/A/62A76ABB-9990-4EFC-A4FE-C7D698DAEB96/9600.16384.WINBLUE_RTM.130821-1623_X64FRE_SERVER_EVAL_EN-US-IRM_SSS_X64FREE_EN-US_DV5.ISO" -var "iso_checksum=7e3f89dbff163e259ca9b0d1f078daafd2fed513" eval-win2012r2-standard.json
virtualbox-iso output will be in this color.

==> virtualbox-iso: Cannot find "Default Guest Additions ISO" in vboxmanage output (or it is empty)
==> virtualbox-iso: Downloading or copying Guest additions checksums
    virtualbox-iso: Downloading or copying: http://download.virtualbox.org/virtualbox/5.0.18/SHA256SUMS
==> virtualbox-iso: Downloading or copying Guest additions
    virtualbox-iso: Downloading or copying: http://download.virtualbox.org/virtualbox/5.0.18/VBoxGuestAdditions_5.0.18.iso
    virtualbox-iso: Download progress: 7%
    virtualbox-iso: Download progress: 99%
    virtualbox-iso: Download progress: 100%
    virtualbox-iso: Download progress: 100%
    virtualbox-iso: Download progress: 100%
    virtualbox-iso: Download progress: 100%
==> virtualbox-iso: Creating floppy disk...
    virtualbox-iso: Copying: floppy/00-run-all-scripts.cmd
    virtualbox-iso: Copying: floppy/01-install-wget.cmd
    virtualbox-iso: Copying: floppy/_download.cmd
    virtualbox-iso: Copying: floppy/_packer_config.cmd
    virtualbox-iso: Copying: floppy/disablewinupdate.bat
    virtualbox-iso: Copying: floppy/eval-win2012r2-standard/Autounattend.xml
    virtualbox-iso: Copying: floppy/fixnetwork.ps1
    virtualbox-iso: Copying: floppy/install-winrm.cmd
    virtualbox-iso: Copying: floppy/oracle-cert.cer
    virtualbox-iso: Copying: floppy/passwordchange.bat
    virtualbox-iso: Copying: floppy/powerconfig.bat
    virtualbox-iso: Copying: floppy/zz-start-sshd.cmd
==> virtualbox-iso: Creating virtual machine...
==> virtualbox-iso: Creating hard drive...
==> virtualbox-iso: Attaching floppy disk...
==> virtualbox-iso: Creating forwarded port mapping for communicator (SSH, WinRM, etc) (host port 4185)
==> virtualbox-iso: Executing custom VBoxManage commands...
    virtualbox-iso: Executing: modifyvm eval-win2012r2-standard --memory 1536
    virtualbox-iso: Executing: modifyvm eval-win2012r2-standard --cpus 1
    virtualbox-iso: Executing: setextradata eval-win2012r2-standard VBoxInternal/CPUM/CMPXCHG16B 1
==> virtualbox-iso: Starting the virtual machine...
==> virtualbox-iso: Waiting 10s for boot...
==> virtualbox-iso: Typing the boot command...
==> virtualbox-iso: Waiting for WinRM to become available...
==> virtualbox-iso: Connected to WinRM!
==> virtualbox-iso: Uploading VirtualBox version info (5.0.18)
==> virtualbox-iso: Uploading VirtualBox guest additions ISO...
==> virtualbox-iso: Provisioning with windows-shell...
==> virtualbox-iso: Provisioning with shell script: script/vagrant.bat
    virtualbox-iso: ==> Creating "C:\Users\vagrant\AppData\Local\Temp\vagrant"
    virtualbox-iso: ==> Downloading "https://raw.githubusercontent.com/mitchellh/vagrant/master/keys/vagrant.pub" to "C:\Users\vagrant\AppData\Local\Temp\vagrant\vagrant.pub"
    virtualbox-iso: WARNING: cannot verify raw.githubusercontent.com's certificate, issued by 'CN=DigiCert SHA2 High Assurance Server CA,OU=www.digicert.com,O=DigiCert Inc,C=US':
    virtualbox-iso: Unable to locally verify the issuer's authority.
    virtualbox-iso: 2016-09-22 13:44:20 URL:https://raw.githubusercontent.com/mitchellh/vagrant/master/keys/vagrant.pub [409/409] -> "C:/Users/vagrant/AppData/Local/Temp/vagrant/vagrant.pub" [1]
    virtualbox-iso: ==> Creating "C:\Users\vagrant\.ssh"
    virtualbox-iso: ==> Adding "C:\Users\vagrant\AppData\Local\Temp\vagrant\vagrant.pub" to "C:\Users\vagrant\.ssh\authorized_keys"
    virtualbox-iso: ==> Disabling account password expiration for user "vagrant"
    virtualbox-iso: Updating property(s) of '\\WIN-80PPKE0JMK0\ROOT\CIMV2:Win32_UserAccount.Domain="WIN-80PPKE0JMK0",Name="vagrant"'
    virtualbox-iso: Property(s) update successful.
    virtualbox-iso:
    virtualbox-iso: Pinging 127.0.0.1 with 32 bytes of data:
    virtualbox-iso: Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
    virtualbox-iso: Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
    virtualbox-iso: Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
    virtualbox-iso: Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
    virtualbox-iso:
    virtualbox-iso: Ping statistics for 127.0.0.1:
    virtualbox-iso: Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
    virtualbox-iso: Approximate round trip times in milli-seconds:
    virtualbox-iso: Minimum = 0ms, Maximum = 0ms, Average = 0ms
    virtualbox-iso: ==> Script exiting with errorlevel 0
==> virtualbox-iso: Provisioning with shell script: script/cmtool.bat
    virtualbox-iso: ==> Building box without a configuration management tool
    virtualbox-iso:
    virtualbox-iso: Pinging 127.0.0.1 with 32 bytes of data:
    virtualbox-iso: Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
    virtualbox-iso: Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
    virtualbox-iso: Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
    virtualbox-iso: Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
    virtualbox-iso:
    virtualbox-iso: Ping statistics for 127.0.0.1:
    virtualbox-iso: Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
    virtualbox-iso: Approximate round trip times in milli-seconds:
    virtualbox-iso: Minimum = 0ms, Maximum = 0ms, Average = 0ms
    virtualbox-iso: ==> Script exiting with errorlevel 0
==> virtualbox-iso: Provisioning with shell script: script/vmtool.bat
    virtualbox-iso: ==> Creating "C:\Users\vagrant\AppData\Local\Temp\sevenzip"
    virtualbox-iso: ==> Downloading "http://www.7-zip.org/a/7z1600-x64.msi" to "C:\Users\vagrant\AppData\Local\Temp\sevenzip\7z1600-x64.msi"
    virtualbox-iso: 2016-09-22 13:44:33 URL:http://d.7-zip.org/a/7z1600-x64.msi [1664000/1664000] -> "C:/Users/vagrant/AppData/Local/Temp/sevenzip/7z1600-x64.msi" [1]
    virtualbox-iso: ==> Installing "C:\Users\vagrant\AppData\Local\Temp\sevenzip\7z1600-x64.msi"
    virtualbox-iso: ==> Copying "C:\Program Files\7-Zip\7z.exe" to "C:\Windows"
    virtualbox-iso: 1 file(s) copied.
    virtualbox-iso: 1 file(s) copied.
    virtualbox-iso: ==> Extracting the VirtualBox Guest Additions installer
    virtualbox-iso:
    virtualbox-iso: 7-Zip [64] 16.00 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-10
    virtualbox-iso:
    virtualbox-iso: Scanning the drive for archives:
    virtualbox-iso: 1 file, 58144768 bytes (56 MiB)
    virtualbox-iso:
    virtualbox-iso: Extracting archive: C:\Users\vagrant\VBoxGuestAdditions.iso
    virtualbox-iso: --
    virtualbox-iso: Path = C:\Users\vagrant\VBoxGuestAdditions.iso
    virtualbox-iso: Type = Iso
    virtualbox-iso: Physical Size = 58144768
    virtualbox-iso: Created = 2016-04-18 06:38:18
    virtualbox-iso: Modified = 2016-04-18 06:38:18
    virtualbox-iso:
    virtualbox-iso: Everything is Ok
    virtualbox-iso:
    virtualbox-iso: Size:       16169336
    virtualbox-iso: Compressed: 58144768
    virtualbox-iso: ==> Installing Oracle certificate to keep install silent
    virtualbox-iso: TrustedPublisher "Trusted Publishers"
    virtualbox-iso: Certificate "Oracle Corporation" added to store.
    virtualbox-iso: CertUtil: -addstore command completed successfully.
    virtualbox-iso: ==> Installing VirtualBox Guest Additions
    virtualbox-iso: ==> Script exiting with errorlevel 0
    virtualbox-iso:
    virtualbox-iso: Pinging 127.0.0.1 with 32 bytes of data:
    virtualbox-iso: Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
    virtualbox-iso: Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
    virtualbox-iso: Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
    virtualbox-iso: Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
    virtualbox-iso: Could Not Find C:\Users\vagrant\AppData\Local\Temp\script.bat-25146.tmp
    virtualbox-iso:
    virtualbox-iso: Ping statistics for 127.0.0.1:
    virtualbox-iso: Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
    virtualbox-iso: Approximate round trip times in milli-seconds:
    virtualbox-iso: Minimum = 0ms, Maximum = 0ms, Average = 0ms
==> virtualbox-iso: Provisioning with shell script: script/clean.bat
    virtualbox-iso: del /f /q /s "C:\Windows\TEMP\DMI7F57.tmp"
    virtualbox-iso: del /f /q /s "C:\Windows\TEMP\winstore.log"
    virtualbox-iso: ==> Cleaning "C:\Users\vagrant\AppData\Local\Temp" directories
    virtualbox-iso: ==> Cleaning "C:\Users\vagrant\AppData\Local\Temp" files
    virtualbox-iso: ==> Cleaning "C:\Windows\TEMP" directories
    virtualbox-iso: ==> Removing potentially corrupt recycle bin
    virtualbox-iso: ==> Cleaning "C:\Windows\TEMP" files
    virtualbox-iso: ==> Cleaning "C:\Users\vagrant"
    virtualbox-iso:
    virtualbox-iso: Pinging 127.0.0.1 with 32 bytes of data:
    virtualbox-iso: Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
    virtualbox-iso: Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
    virtualbox-iso: Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
    virtualbox-iso: Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
    virtualbox-iso:
    virtualbox-iso: Ping statistics for 127.0.0.1:
    virtualbox-iso: Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
    virtualbox-iso: Approximate round trip times in milli-seconds:
    virtualbox-iso: Minimum = 0ms, Maximum = 0ms, Average = 0ms
    virtualbox-iso: ==> Script exiting with errorlevel 0
==> virtualbox-iso: Provisioning with shell script: script/ultradefrag.bat
    virtualbox-iso: ==> Creating "C:\Users\vagrant\AppData\Local\Temp\ultradefrag"
    virtualbox-iso: ==> Downloading "http://downloads.sourceforge.net/ultradefrag/ultradefrag-portable-7.0.1.bin.amd64.zip" to "C:\Users\vagrant\AppData\Local\Temp\ultradefrag\ultradefrag-portable-7.0.1.bin.amd64.zip"
    virtualbox-iso: http://downloads.sourceforge.net/ultradefrag/ultradefrag-portable-7.0.1.bin.amd64.zip:
    virtualbox-iso: 2016-09-22 13:45:01 ERROR 404: Not Found.
    virtualbox-iso: ==> Unzipping "C:\Users\vagrant\AppData\Local\Temp\ultradefrag\ultradefrag-portable-7.0.1.bin.amd64.zip" to "C:\Users\vagrant\AppData\Local\Temp\ultradefrag"
    virtualbox-iso:
    virtualbox-iso: 7-Zip [64] 16.00 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-10
    virtualbox-iso:
    virtualbox-iso: Scanning the drive for archives:
    virtualbox-iso: 1 file, 3596965 bytes (3513 KiB)
    virtualbox-iso:
    virtualbox-iso: Extracting archive: C:\Users\vagrant\AppData\Local\Temp\ultradefrag\ultradefrag-portable-7.0.1.bin.amd64.zip
    virtualbox-iso: --
    virtualbox-iso: Path = C:\Users\vagrant\AppData\Local\Temp\ultradefrag\ultradefrag-portable-7.0.1.bin.amd64.zip
    virtualbox-iso: Type = zip
    virtualbox-iso: Physical Size = 3596965
    virtualbox-iso:
    virtualbox-iso: Everything is Ok
    virtualbox-iso:
    virtualbox-iso: Files: 4
    virtualbox-iso: Size:       2753024
    virtualbox-iso: Compressed: 3596965
    virtualbox-iso: ==> Running UltraDefrag on C:
    virtualbox-iso: UltraDefrag 7.0.1, Copyright (c) UltraDefrag Development Team, 2007-2016.
    virtualbox-iso: UltraDefrag comes with ABSOLUTELY NO WARRANTY. This is free software,
    virtualbox-iso: and you are welcome to redistribute it under certain conditions.
    virtualbox-iso:
    virtualbox-iso: C: defrag:   100.00% complete, 7 passes needed, fragmented/total = 4/75370
    virtualbox-iso: ==> Removing "C:\Users\vagrant\AppData\Local\Temp\ultradefrag"
    virtualbox-iso:
    virtualbox-iso: Pinging 127.0.0.1 with 32 bytes of data:
    virtualbox-iso: Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
    virtualbox-iso: Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
    virtualbox-iso: Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
    virtualbox-iso: Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
    virtualbox-iso:
    virtualbox-iso: Ping statistics for 127.0.0.1:
    virtualbox-iso: Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
    virtualbox-iso: Approximate round trip times in milli-seconds:
    virtualbox-iso: Minimum = 0ms, Maximum = 0ms, Average = 0ms
    virtualbox-iso: ==> Script exiting with errorlevel 0
==> virtualbox-iso: Provisioning with shell script: script/uninstall-7zip.bat
    virtualbox-iso: ==> Uninstalling 7zip
    virtualbox-iso: ==> WARNING: Directory not found: "C:\Users\vagrant\AppData\Local\Temp\sevenzip"
    virtualbox-iso: ==> Removing "C:\Program Files\7-Zip"
    virtualbox-iso:
    virtualbox-iso: Pinging 127.0.0.1 with 32 bytes of data:
    virtualbox-iso: Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
    virtualbox-iso: Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
    virtualbox-iso: Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
    virtualbox-iso: Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
    virtualbox-iso:
    virtualbox-iso: Ping statistics for 127.0.0.1:
    virtualbox-iso: Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
    virtualbox-iso: Approximate round trip times in milli-seconds:
    virtualbox-iso: Minimum = 0ms, Maximum = 0ms, Average = 0ms
    virtualbox-iso: ==> Script exiting with errorlevel 0
==> virtualbox-iso: Provisioning with shell script: script/sdelete.bat
    virtualbox-iso: ==> Creating "C:\Users\vagrant\AppData\Local\Temp\sdelete"
    virtualbox-iso: ==> Downloading "http://live.sysinternals.com/sdelete.exe" to "C:\Users\vagrant\AppData\Local\Temp\sdelete\sdelete.exe"
    virtualbox-iso: WARNING: cannot verify live.sysinternals.com's certificate, issued by 'CN=Microsoft IT SSL SHA2,OU=Microsoft IT,O=Microsoft Corporation,L=Redmond,ST=Washington,C=US':
    virtualbox-iso: Unable to locally verify the issuer's authority.
    virtualbox-iso: The operation completed successfully.
    virtualbox-iso: 2016-09-22 13:59:14 URL:https://live.sysinternals.com/sdelete.exe [151200/151200] -> "C:/Users/vagrant/AppData/Local/Temp/sdelete/sdelete.exe" [1]
    virtualbox-iso: ==> Running SDelete on C:
    virtualbox-iso:
    virtualbox-iso: SDelete v2.0 - Secure file delete
    virtualbox-iso: Copyright (C) 1999-2016 Mark Russinovich
    virtualbox-iso: Sysinternals - www.sysinternals.com
    virtualbox-iso:
    virtualbox-iso: SDelete is set for 1 pass.
```

#### Adding Box to `vagrant`


```ruby
┌─[ahmed][zubair-HP-ProBook][±][master ✓][~/work/windows]
└─▪ cd box/virtualbox/
┌─[ahmed][zubair-HP-ProBook][±][master ✓][~/work/windows/box/virtualbox]
└─▪ ls
eval-win2012r2-standard-nocm-1.0.4.box
┌─[ahmed][zubair-HP-ProBook][±][master ✓][~/work/windows/box/virtualbox]
└─▪ vagrant box add windows-2012r2 eval-win2012r2-standard-nocm-1.0.4.box
```

#### Update the `.kitchen.yml` on your cookbook.

```ruby
---
driver:
  name: vagrant

provisioner:
  name: chef_zero

platforms:
  - name: windows-2012r2

suites:
  - name: default
    run_list:
      - recipe[starter-windows-cookbook::default]
```

#### List VM


Command

``` ruby
kitchen list
```

#### VM Details

``` ruby
┌─[ahmed][zubair-HP-ProBook][±][master U:3 ?:2 ✗][~/work/chef-repo/cookbooks/nagios_nrpe_deploy]
└─▪ kitchen list
Instance                Driver   Provisioner  Verifier  Transport  Last Action
windows-2012r2          Vagrant  ChefZero     Busser    Winrm      <Not Created>
```

#### Testing Windows VM - Using command below.

``` ruby
kitchen test
```

We are done !!!! Enjoy Windows Testing.

### Important Links.

- [`http://kitchen.ci/blog/test-kitchen-windows-test-flight-with-vagrant/`](http://kitchen.ci/blog/test-kitchen-windows-test-flight-with-vagrant/)
- [`http://rriifftt.hatenablog.com/entry/2016/06/05/215712`](http://rriifftt.hatenablog.com/entry/2016/06/05/215712)
