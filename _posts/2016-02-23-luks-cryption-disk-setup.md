---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: LUKS Disk encryption for CentOS 6.6/RHEL 6
category: ['Linux', 'Centos', 'Luks']
tags: ['centos', 'linux', 'luks', 'disk', 'encryption']
---

Linux Unified Key Setup-on-disk-format (or LUKS) allows you to encrypt partitions on your Linux computer. 
This is particularly important when it comes to mobile computers and removable media. 
LUKS allows multiple user keys to decrypt a master key, which is used for the bulk encryption of the partition.


What LUKS does ? [from redhat site](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Security_Guide/sec-Encryption.html).

1. LUKS encrypts entire block devices and is therefore well-suited for protecting the contents of mobile devices such as removable storage media or laptop disk drives.
2. The underlying contents of the encrypted block device are arbitrary. This makes it useful for encrypting swap devices. This can also be useful with certain databases that use specially formatted block devices for data storage.
3. LUKS uses the existing device mapper kernel subsystem.
4. LUKS provides passphrase strengthening which protects against dictionary attacks.
5. LUKS devices contain multiple key slots, allowing users to add backup keys or passphrases.

What LUKS does not do: [from redhat site](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Security_Guide/sec-Encryption.html).

1. LUKS is not well-suited for applications requiring many (more than eight) users to have distinct access keys to the same device.
2. LUKS is not well-suited for applications requiring file-level encryption.

Lets begin.

Lets assume that we have a drive partition which is ready to be encrypted.
To create a partition we can get more information [here](http://zubayr.github.io/raid10-format-parted-blog/). Which is an older post which gives details about [formatting and mount disk on a RAID partition](http://zubayr.github.io/raid10-format-parted-blog/). But the steps are similar when we add a new drive.


We have already mounted the partition `/dev/sdb1` to `/crypt_fs`.

    [root@localhost ahmed]# df -k
    Filesystem     1K-blocks    Used Available Use% Mounted on
    /dev/sda2       18208184 6542312  10734288  38% /
    tmpfs             506144     228    505916   1% /dev/shm
    /dev/sda1         289293   28459    245474  11% /boot
    /dev/sdb1        2030444    3076   1922564   1% /crypt_fs

####  Step 1. Lets `unmount` the partition.
    
    [root@localhost ahmed]# umount /crypt_fs

####  Step 1.1. Check `cryptsetup` help
    
    [root@localhost /]# cryptsetup --help
    cryptsetup 1.2.0
    Usage: cryptsetup [OPTION...] <action> <action-specific>]
      --version                       Print package version
      -v, --verbose                   Shows more detailed error messages
      --debug                         Show debug messages
      -c, --cipher=STRING             The cipher used to encrypt the disk (see /proc/crypto)
      -h, --hash=STRING               The hash used to create the encryption key from the passphrase
      -y, --verify-passphrase         Verifies the passphrase by asking for it twice
      -d, --key-file=STRING           Read the key from a file.
      --master-key-file=STRING        Read the volume (master) key from file.
      --dump-master-key               Dump volume (master) key instead of keyslots info.
      -s, --key-size=BITS             The size of the encryption key
      -l, --keyfile-size=bytes        Limits the read from keyfile
      --new-keyfile-size=bytes        Limits the read from newly added keyfile
      -S, --key-slot=INT              Slot number for new key (default is first free)
      -b, --size=SECTORS              The size of the device
      -o, --offset=SECTORS            The start offset in the backend device
      -p, --skip=SECTORS              How many sectors of the encrypted data to skip at the beginning
      -r, --readonly                  Create a readonly mapping
      -i, --iter-time=msecs           PBKDF2 iteration time for LUKS (in ms)
      -q, --batch-mode                Do not ask for confirmation
      -t, --timeout=secs              Timeout for interactive passphrase prompt (in seconds)
      -T, --tries=INT                 How often the input of the passphrase can be retried
      --align-payload=SECTORS         Align payload at <n> sector boundaries - for luksFormat
      --header-backup-file=STRING     File with LUKS header and keyslots backup.
      --use-random                    Use /dev/random for generating volume key.
      --use-urandom                   Use /dev/urandom for generating volume key.
      --uuid=STRING                   UUID for device to use.

    Help options:
      -?, --help                      Show this help message
      --usage                         Display brief usage

    <action> is one of:
            create <name> <device> - create device
            remove <name> - remove device
            resize <name> - resize active device
            status <name> - show device status
            luksFormat <device> [<new key file>] - formats a LUKS device
            luksOpen <device> <name>  - open LUKS device as mapping <name>
            luksAddKey <device> [<new key file>] - add key to LUKS device
            luksRemoveKey <device> [<key file>] - removes supplied key or key file from LUKS device
            luksKillSlot <device> <key slot> - wipes key with number <key slot> from LUKS device
            luksUUID <device> - print UUID of LUKS device
            isLuks <device> - tests <device> for LUKS partition header
            luksClose <name> - remove LUKS mapping
            luksDump <device> - dump LUKS partition information
            luksSuspend <device> - Suspend LUKS device and wipe key (all IOs are frozen).
            luksResume <device> - Resume suspended LUKS device.
            luksHeaderBackup <device> - Backup LUKS device header and keyslots
            luksHeaderRestore <device> - Restore LUKS device header and keyslots

    <name> is the device to create under /dev/mapper
    <device> is the encrypted device
    <key slot> is the LUKS key slot number to modify
    <key file> optional key file for the new key for luksAddKey action

    Default compiled-in device cipher parameters:
            plain: aes-cbc-essiv:sha256, Key: 256 bits, Password hashing: ripemd160
            LUKS1: aes-cbc-essiv:sha256, Key: 256 bits, LUKS header hashing: sha1, RNG: /dev/urandom
    
    
####  Step 2. Format the partition using LUKS which will overwrite any data which is present on it. 

    [root@localhost ahmed]# cryptsetup luksFormat /dev/sdb1

    WARNING!
    ========
    This will overwrite data on /dev/sdb1 irrevocably.

    Are you sure? (Type uppercase yes): YES
    Enter LUKS passphrase:
    Verify passphrase:


####  Step 3. Open the partition and mount. 

Checking disk.

    [root@localhost crypt_fs]# df -k
    Filesystem     1K-blocks    Used Available Use% Mounted on
    /dev/sda2       18208184 6542336  10734264  38% /
    tmpfs             506144     228    505916   1% /dev/shm
    /dev/sda1         289293   28459    245474  11% /boot

Open disk using `cryptsetup` to `crypt_fs`, `crypt_fs` will be present in `/dev/mapper/crypt_fs`.    

    [root@localhost /]# cryptsetup luksOpen /dev/sdb1 crypt_fs
    Enter passphrase for /dev/sdb1:

####  Step 4. Creating and keyfile and store it on the disk.

Create a keyfile.

    [root@localhost /]# dd if=/dev/urandom of=/root/keyfile bs=1024 count=4
    4+0 records in
    4+0 records out
    4096 bytes (4.1 kB) copied, 0.000893462 s, 4.6 MB/s

Change permissions to `0400`, NO ONE except root should be able to access this.

    [root@localhost /]# chmod 0400 /root/keyfile
    

####  Step 5. Adding key to the disk.
    
    [root@localhost /]# cryptsetup luksAddKey /dev/sdb1 /root/keyfile
    Enter any passphrase:

####  Step 6. `mkfs.ext4` setting file system.
    
    [root@localhost /]# mkfs.ext4 /dev/mapper/crypt_fs
    mke2fs 1.41.12 (17-May-2010)
    Filesystem label=
    OS type: Linux
    Block size=4096 (log=2)
    Fragment size=4096 (log=2)
    Stride=0 blocks, Stripe width=0 blocks
    131072 inodes, 523527 blocks
    26176 blocks (5.00%) reserved for the super user
    First data block=0
    Maximum filesystem blocks=536870912
    16 block groups
    32768 blocks per group, 32768 fragments per group
    8192 inodes per group
    Superblock backups stored on blocks:
            32768, 98304, 163840, 229376, 294912

    Writing inode tables: done
    Creating journal (8192 blocks): done
    Writing superblocks and filesystem accounting information: done

    This filesystem will be automatically checked every 28 mounts or
    180 days, whichever comes first.  Use tune2fs -c or -i to override.

####  Step 7. mounting the `/dev/mapper/crypt_fs` disk to mount-point `crypt_fs`
    
Create a mount point if it is not present, but we already have an earlier mount point called `/crypt_fs` we will use the same thing. (DONT NOT get confused with the `crypt_fs` which is in `/dev/mapper/crypt_fs`, these two are different as the command below will make it clear)

    [root@localhost /]# mount /dev/mapper/crypt_fs /crypt_fs
    [root@localhost /]# df
    Filesystem           1K-blocks    Used Available Use% Mounted on
    /dev/sda2             18208184 6542344  10734256  38% /
    tmpfs                   506144     228    505916   1% /dev/shm
    /dev/sda1               289293   28459    245474  11% /boot
    /dev/mapper/crypt_fs   2028396    3072   1920620   1% /crypt_fs

####  Step 8. Creating `crypttab` file.    

Add the below line to the `crypttab` file.
 
    [root@localhost /]# echo "crypt_fs /dev/sdb1 /root/keyfile luks" >> /etc/crypttab
    [root@localhost /]# cat /etc/crypttab
    crypt_fs /dev/sdb1 /root/keyfile luks
    
    
####  Step 9. Update `/etc/fstab` file.

Add the line below to the `/etc/fstab` file.

    /dev/mapper/crypt_fs    /crypt_fs               ext4    defaults        1 2

Here is the output of the file.
    
    [root@localhost ahmed]# cat /etc/fstab

    # 
    #  /etc/fstab
    #  Created by anaconda on Fri May  8 04:35:06 2015
    # 
    #  Accessible filesystems, by reference, are maintained under '/dev/disk'
    #  See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
    # 
    UUID=476bdfc5-b4cc-43a2-86c4-9d7aace3385a /                       ext4    defaults        1 1
    UUID=8bf10a61-d2de-4cda-bd1d-d5e70aaea1b5 /boot                   ext4    defaults        1 2
    UUID=c5c5584d-c6a4-467b-b26c-b2bac80c7165 swap                    swap    defaults        0 0
    tmpfs                   /dev/shm                tmpfs   defaults        0 0
    devpts                  /dev/pts                devpts  gid=5,mode=620  0 0
    sysfs                   /sys                    sysfs   defaults        0 0
    proc                    /proc                   proc    defaults        0 0
    /dev/mapper/crypt_fs    /crypt_fs               ext4    defaults        1 2
    [root@localhost ahmed]#


####  Step 10. Reboot the server to see if the disk mounts without a password.

    [root@localhost /]# reboot

    Broadcast message from ahmed@localhost.localdomain
            (/dev/pts/1) at 17:16 ...

    The system is going down for reboot NOW!
    [root@localhost /]#
    Using username "ahmed".
    ahmed@192.168.126.131's password:
    Last login: Tue Feb 23 17:14:02 2016 from 192.168.126.1
    This is BASH 4.1     - DISPLAY on

    Tue Feb 23 17:19:26 IST 2016
    [ahmed@localhost ~]$ sudo su
    [root@localhost ahmed]# df -h
    Filesystem            Size  Used Avail Use% Mounted on
    /dev/sda2              18G  6.3G   11G  38% /
    tmpfs                 495M   80K  495M   1% /dev/shm
    /dev/sda1             283M   28M  240M  11% /boot
    /dev/mapper/crypt_fs  2.0G  3.0M  1.9G   1% /crypt_fs
    [root@localhost ahmed]# cryptsetup luksDump /dev/mapper/crypt_fs
    Device /dev/mapper/crypt_fs is not a valid LUKS device.

####  Step 11. Checking keys.

Currently we are using 2 slots on the disk.

1. Password based key, which we have on the beginning of the setup.
2. Is the `keyfile` which we inserted into the disk using the `luksAddKey` command.

Here is the dump. 

    [root@localhost ahmed]# cryptsetup luksDump /dev/sdb1
    LUKS header information for /dev/sdb1

    Version:        1
    Cipher name:    aes
    Cipher mode:    cbc-essiv:sha256
    Hash spec:      sha1
    Payload offset: 4096
    MK bits:        256
    MK digest:      6b a5 e6 1e bd 2d 0c e3 6e 43 af 46 e9 5e 9b 40 59 fc 10 89
    MK salt:        8b 3c 4e 99 81 2e e5 cd 2e 9f 61 f9 d2 5e 13 9a
                    13 71 0a e7 65 ff c4 b7 5c 4c 5a 15 04 7f 22 5d
    MK iterations:  138875
    UUID:           530f17e2-ea3c-442d-9cfc-e9da5f72630d

    Key Slot 0: ENABLED
            Iterations:             555505
            Salt:                   8a 96 63 b8 21 e1 d9 1a e6 4c 7e e8 2b 02 b5 04
                                    e8 5f be ac e2 d9 3f 48 4c b9 0b 74 dd c3 09 38
            Key material offset:    8
            AF stripes:             4000
    Key Slot 1: ENABLED
            Iterations:             745495
            Salt:                   60 dc 17 b3 bd 27 19 18 48 e8 22 9e 96 d6 b9 e9
                                    95 f3 71 06 bf 3e e4 73 e5 d7 23 ac 3b 1a 7a b0
            Key material offset:    264
            AF stripes:             4000
    Key Slot 2: DISABLED
    Key Slot 3: DISABLED
    Key Slot 4: DISABLED
    Key Slot 5: DISABLED
    Key Slot 6: DISABLED
    Key Slot 7: DISABLED

Setup complete. Now the disk `/dev/sdb1` is encrypted and ready to use.
    