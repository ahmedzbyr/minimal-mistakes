---
title: How To Configure Swappiness
category: ['Hadoop', 'Linux']
tags: ['hadoop', 'linux', 'swappiness', 'swap', 'sysctl', 'config']
---

Swappiness is a Linux kernel parameter that controls the relative weight given to swapping out runtime memory, as opposed to dropping pages from the system page cache. Swappiness can be set to values between 0 and 100 inclusive. A low value causes the kernel to avoid swapping, a higher value causes the kernel to try to use swap space. The default value is 60, and for most desktop systems, setting it to 100 may affect the overall performance, whereas setting it lower (even 0) may decrease response latency.

    Value	                Strategy
    vm.swappiness = 0	    The kernel will swap only to avoid an out of memory condition. 
                                See the "VM Sysctl documentation".
    vm.swappiness = 1	    Kernel version 3.5 and over, as well as kernel version 2.6.32-303 
                                and over: Minimum amount of swapping without disabling it entirely.
    vm.swappiness = 10	    This value is sometimes recommended to improve performance 
                                when sufficient memory exists in a system.
    vm.swappiness = 60	    The default value.
    vm.swappiness = 100     The kernel will swap aggressively.

With kernel version `3.5` and over, as well as kernel version `2.6.32-303` and over, it is likely better to use `1` for cases where `0` used to be optimal.
To temporarily set the swappiness in Linux, write the desired value (e.g. 10) to `/proc/sys/vm/swappiness` using the following command, running as root user.   

    #  Set the swappiness value as root
    echo 10 > /proc/sys/vm/swappiness

    #  Alternatively, run this 
    sysctl -w vm.swappiness=10

    #  Verify the change
    cat /proc/sys/vm/swappiness
    10

    #  Alternatively, verify the change
    sysctl vm.swappiness
    vm.swappiness = 10
    
To find the current swappiness settings, type:

	cat /proc/sys/vm/swappiness
	60

Swapiness can be a value from 0 to 100. 

1. Swappiness near 100 means that the operating system will swap often and usually, too soon. 
2. Although swap provides extra resources, RAM is much faster than swap space. Any time something is moved from RAM to swap, it slows down.

A swappiness value of 0 means that the operating will only rely on swap when it absolutely needs to. We can adjust the swappiness with the sysctl command:

	sysctl vm.swappiness=10
	vm.swappiness=10

If we check the system swappiness again, we can confirm that the setting was applied:

	cat /proc/sys/vm/swappiness
	10

To make changes permanent, you can add the setting to the /etc/sysctl.conf file:

	sudo nano /etc/sysctl.conf

Add the below line.
	
	#  Search for the vm.swappiness setting.  Uncomment and change it as necessary.
	vm.swappiness=10