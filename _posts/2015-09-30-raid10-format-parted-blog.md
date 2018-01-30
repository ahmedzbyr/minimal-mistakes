---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: Mounting RAID10 using `parted`.
category: ['Linux', 'Raid']
tags: ['linux', 'gparted', 'parted', 'mount', 'raid']
---

GUID Partition Table (GPT) is a standard for the layout of the partition table on a physical hard disk, using globally unique identifiers (GUID). Although it forms a part of the Unified Extensible Firmware Interface (UEFI) standard (Unified EFI Forum proposed replacement for the PC BIOS), it is also used on some BIOS systems because of the limitations of master boot record (MBR) partition tables, which use 32 bits for storing logical block addresses (LBA) and size information.

Below is the image of how partition is divided. (courtesy from [wikipedia](https://en.wikipedia.org/wiki/File:GUID_Partition_Table_Scheme.svg))


![image](http://zubayr.github.io/images/guid_partition.png)



##  Logging into the server

First lets check the `fdisk` partition to see how much space we have on the server.

	Using username "root".
	root@192.168.100.44's password:
	Last login: Wed Sep 30 13:22:13 2015 from 192.168.100.2
	[root@my-server ~]# fdisk -l /dev/sdb

	WARNING: GPT (GUID Partition Table) detected on '/dev/sdb'! 
                                        The util fdisk doesn't support GPT. Use GNU Parted.


	Disk /dev/sdb: 13196.0 GB, 13196018581504 bytes
	255 heads, 63 sectors/track, 1604324 cylinders
	Units = cylinders of 16065 * 512 = 8225280 bytes
	Sector size (logical/physical): 512 bytes / 512 bytes
	I/O size (minimum/optimal): 512 bytes / 512 bytes
	Disk identifier: 0x00000000

	   Device Boot      Start         End      Blocks   Id  System
	/dev/sdb1               1      267350  2147483647+  ee  GPT


Checking disk.	
	
	[root@my-server ~]# df -h
	Filesystem            Size  Used Avail Use% Mounted on
	/dev/mapper/VG-LV_ROOT
						   96G  1.9G   90G   2% /
	tmpfs                 127G     0  127G   0% /dev/shm
	/dev/sda2             976M   32M  894M   4% /boot
	/dev/sda1            1022M  276K 1022M   1% /boot/efi
	/dev/mapper/VG-LV_HOME
						  976M  1.3M  924M   1% /home
	/dev/mapper/VG-LV_VAR
						  998G  1.5G  946G   1% /var

Let us start partitioning the RAID on the server. 
 						  
	[root@my-server ~]# parted /dev/sdb
	GNU Parted 2.1
	Using /dev/sdb
	Welcome to GNU Parted! Type 'help' to view a list of commands.
	(parted) print
	Model: DELL PERC H730 Mini (scsi)
	Disk /dev/sdb: 13196GB
	Sector size (logical/physical): 512B/512B
	Partition Table: gpt
	
	Number  Start  End  Size  File system  Name  Flags
	
	(parted) mklabel gpt
	Warning: The existing disk label on /dev/sdb will be destroyed and 
                                    all data on this disk will be lost. Do you want to continue?
	Yes/No? Yes
	(parted) unit GB
	(parted) mkpart primary 1MB 13196GB
	(parted) print
	Model: DELL PERC H730 Mini (scsi)
	Disk /dev/sdb: 13196GB
	Sector size (logical/physical): 512B/512B
	Partition Table: gpt
	
	Number  Start   End      Size     File system  Name     Flags
	 1      0.00GB  13196GB  13196GB  ext4         primary
	
	(parted) quit


partition is created. Now we are going to format it using `ext4`.


	[root@my-server ~]# mkfs.ext4 /dev/sdb1
	mke2fs 1.41.12 (17-May-2010)
	Filesystem label=
	OS type: Linux
	Block size=4096 (log=2)
	Fragment size=4096 (log=2)
	Stride=0 blocks, Stripe width=0 blocks
	732422144 inodes, 2929687296 blocks
	146484364 blocks (5.00%) reserved for the super user
	First data block=0
	Maximum filesystem blocks=4294967296
	89407 block groups
	32768 blocks per group, 32768 fragments per group
	8192 inodes per group
	Superblock backups stored on blocks:
			32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
			4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968,
			102400000, 214990848, 512000000, 550731776, 644972544, 1934917632,
			2560000000

	Writing inode tables: done
	Creating journal (32768 blocks): done
	Writing superblocks and filesystem accounting information: done

	This filesystem will be automatically checked every 22 mounts or
	180 days, whichever comes first.  Use tune2fs -c or -i to override.
	[root@my-server ~]#
	[root@my-server ~]#

Checking for the partition is ready. Now we need to mount it.	
	
	[root@my-server ~]# lsblk
	NAME                  MAJ:MIN RM    SIZE RO TYPE MOUNTPOINT
	sda                     8:0    0    1.1T  0 disk
	├─sda1                  8:1    0      1G  0 part /boot/efi
	├─sda2                  8:2    0      1G  0 part /boot
	└─sda3                  8:3    0    1.1T  0 part
	  ├─VG-LV_ROOT (dm-0) 253:0    0   97.7G  0 lvm  /
	  ├─VG-LV_SWAP (dm-1) 253:1    0      3G  0 lvm  [SWAP]
	  ├─VG-LV_VAR (dm-2)  253:2    0 1013.6G  0 lvm  /var
	  └─VG-LV_HOME (dm-3) 253:3    0      1G  0 lvm  /home
	sdb                     8:16   0     12T  0 disk
	└─sdb1                  8:17   0     12T  0 part /data


Creating a mount point.

	[root@my-server ~]# mkdir /data	

Updating `/etc/fstab`. Add the below line to `/etc/fstab`.
	
	#------------------------------------------------------------
	#  drive	  |	 dir  |	fs-type	 |	options     | dump  |  pass
	#------------------------------------------------------------
	/dev/sdb1   /data   ext4    	defaults        0 		0

Here are more details about what each columns mean.



1. **file system** : The partition or storage device to be mounted.
2. **dir** : The mountpoint where <file system> is mounted to.
3. **fs-type** : The file system type of the partition or storage device to be mounted. Many different file systems are supported: `ext2`, `ext3`, `ext4`, `btrfs`, `reiserfs`, `xfs`, `jfs`, `smbfs`, `iso9660`, `vfat`, `ntfs`, `swap` and `auto`. The auto type lets the mount command guess what type of file system is used. This is useful for optical media (CD/DVD).
4. **options** : Mount options of the filesystem to be used. See the mount man page. Please note that some options are specific to filesystems; to discover them see below in the aforementioned mount man page.
5. **dump** : Used by the dump utility to decide when to make a backup. Dump checks the entry and uses the number to decide if a file system should be backed up. Possible entries are 0 and 1. If 0, dump will ignore the file system; if 1, dump will make a backup. Most users will not have dump installed, so they should put 0 for the dump entry.
6. **pass** : Used by `fsck` to decide which order filesystems are to be checked. Possible entries are 0, 1 and 2. The root file system should have the highest priority 1 (unless its type is `btrfs`, in which case this field should be 0) - all other file systems you want to have checked should have a 2. File systems with a value 0 will not be checked by the `fsck` utility.	
	
Here is how the contents look like.
	
	[root@my-server ~]# cat /etc/fstab
	# 
	#  /etc/fstab
	#  Created by anaconda on Tue May 19 15:57:32 2015
	# 
	#  Accessible filesystems, by reference, are maintained under '/dev/disk'
	#  See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
	# 
	/dev/mapper/VG-LV_ROOT  /                       ext4    defaults        1 1
	UUID=4185b123-5123-45ca-b123-de6d1da123e2 /boot                   ext4    defaults        1 2
	UUID=EB97-DBDC          /boot/efi               vfat    umask=0077,shortname=winnt 0 0
	/dev/mapper/VG-LV_HOME  /home                   ext4    defaults        1 2
	/dev/mapper/VG-LV_VAR   /var                    ext4    defaults        1 2
	/dev/mapper/VG-LV_SWAP  swap                    swap    defaults        0 0
	tmpfs                   /dev/shm                tmpfs   defaults        0 0
	devpts                  /dev/pts                devpts  gid=5,mode=620  0 0
	sysfs                   /sys                    sysfs   defaults        0 0
	proc                    /proc                   proc    defaults        0 0
	/dev/sdb1               /data                   ext4    defaults        0 0
	[root@my-server ~]#

Executing `mount -a` to load the `fstab` entries. 
	
	[root@my-server ~]# mount -a 

Display mount entries.

	[root@my-server ~]# mount
	/dev/mapper/VG-LV_ROOT on / type ext4 (rw)
	proc on /proc type proc (rw)
	sysfs on /sys type sysfs (rw)
	devpts on /dev/pts type devpts (rw,gid=5,mode=620)
	tmpfs on /dev/shm type tmpfs (rw)
	/dev/sda2 on /boot type ext4 (rw)
	/dev/sda1 on /boot/efi type vfat (rw,umask=0077,shortname=winnt)
	/dev/mapper/VG-LV_HOME on /home type ext4 (rw)
	/dev/mapper/VG-LV_VAR on /var type ext4 (rw)
	none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)
	/dev/sdb1 on /data type ext4 (rw)
	
Checking for directory mounting. 
	
	[root@my-server ~]# df -h
	Filesystem            Size  Used Avail Use% Mounted on
	/dev/mapper/VG-LV_ROOT
						   96G  1.9G   90G   2% /
	tmpfs                 127G     0  127G   0% /dev/shm
	/dev/sda2             976M   32M  894M   4% /boot
	/dev/sda1            1022M  276K 1022M   1% /boot/efi
	/dev/mapper/VG-LV_HOME
						  976M  1.3M  924M   1% /home
	/dev/mapper/VG-LV_VAR
						  998G  1.5G  946G   1% /var
	/dev/sdb1              13T   31M   13T   1% /data
	[root@my-server ~]#

Now we are all good.


Important Links : 

[Redhat](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Storage_Administration_Guide/s2-disk-storage-parted-create-part.html)

[Cyber Citi](http://www.cyberciti.biz/tips/fdisk-unable-to-create-partition-greater-2tb.html)