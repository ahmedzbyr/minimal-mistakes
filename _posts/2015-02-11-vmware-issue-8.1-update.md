---
title: VMware Workstation 10 Error `Not enough physical memory is available to power this virtual machine`
category: ['Windows', 'Vmware']
tags: ['windows', 'vmware', 'error', 'virtual-machine']
---

If you are using VMWare Workstation (or VMWare player) on Windows 8.1 and have just update Windows, specifically KB2995388, you may receive this error message when attempt to start a virtual machine.

This issues starts after an update.

## Issue on Windows 8.1
 
You attempt to start a virtual machine running on Workstation 10 and you get a message that you have no memory available to run the vm.

**Message** : `Not enough physical memory is available to power this virtual machine with its configured settings.`

## Solution

* Shut down all running virtual machines
* Close VMware Workstation.
* Open Command prompt in Admin mode.

opening `config.ini` file using nodepad from `command prompt`.

	c:\ProgramData\VMware\VMware Workstation> notepad config.ini


* Open `config.ini` located at `C:\ProgramData\VMware\VMware Workstation`
* Insert this line:     `vmmon.disableHostParameters = TRUE`
* Save and close file.
* Reboot Windows.


## Error Message

![alt text](http://ctobob.com/wp-content/uploads/2014/12/vmware-memory.jpg "Not Enough Memory")


###  Useful Links

[More info](http://ctobob.com/2014/12/04/vmware-workstation-10-error-not-enough-physical-memory-available-power-virtual-machine/)