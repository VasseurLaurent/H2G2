# Install a new hard disk 

**Date : 26/05/2019**

**Difficulty:  Easy**

This notes has beend tested on a Ubuntu Server 18.04 

## Introduction 

In this wiki, I will present how to setup a new hard disk : 

- Add the label ``GPT`` 
- Create a partition 
- Install the file system 
- Mount the file system 

## Add the label GPT

First of all, let's check if our new hard disk is ready and doesn't contain any partition : 

```
fdisk -l
```
Let's check the result : 

```
Disk /dev/sdb: 5 GiB, 5368709120 bytes, 10485760 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

```

This command uses the ``fdisk`` tool to check disk and partitions. We can see above that our disk is a ``5 GiB disk`` and its name is ``/dev/sdb``. Moreover, there is no partition (we don't see partition with name ``/dev/sdb1`` , ``/dev/sdb2`` ...)`

Let's check if the disk has a partition table : 

```
parted /dev/sdb
```
It starts the ``parted`` prompt tool. 
If we use the command ``print`` to display the partition table : 

```
(parted) print 
```
```
Error: /dev/sdb: unrecognised disk label
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sdb: 5369MB
Sector size (logical/physical): 512B/512B
Partition Table: unknown
Disk Flags:
```
The ``parted`` tool cannot read the partition table because the label is unrecognised. If we want to add partitions, we need to chose our label. 

To choose between ``MBR`` and ``GPT``, please refer to the wiki page : 

In this example, we chose GPT : 

```
(parted) mklabel gpt 
```
Let's check with another ``print`` command : 

```
(parted) print 
```
```
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sdb: 5369MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start  End  Size  File system  Name  Flags

```

As you can see, the ``parted`` tool is now able to read the partition table. However, the disk doesn't contain any partition yet. 

## Create a partition

Let's create our first partition with the gdisk tool : 

```
gdisk /dev/sdb

```
Add a new partition with the command "n"
```
Command (? for help): n
```

Choose the number of partition, in this example, I will choose ``1`` : 

```
Partition number (1-128, default 1): 1
```
You can notice that there is 128 partitions possible according to GPT caracteristics 

Now, choose the first sector, I will let it by default : 

```
First sector (34-10485726, default = 34) or {+-}size{KMGTP}:

```

Now choose the last sector. Notice that the difference between the first and the last sector is the size of your disk, for this example, I will choose a size of 300M :

```
Last sector (34-10485726, default = 10485726) or {+-}size{KMGTP}: +300M
```
Now you can choose the type of filesystem according to their code. I will let 8300 because it fits the ext4 needs. If you need more options, use the ``L`` command to check codes (for xfs for example) : 

```
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300):
```
That's it, we have created our first partition. We can now check it with the command ``print`` : 

```
Command (? for help): print
Disk /dev/sdb: 10485760 sectors, 5.0 GiB
Model: VBOX HARDDISK
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 5C5371AA-C7AB-4893-93C3-EC4F18EF5839
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 10485726
Partitions will be aligned on 2-sector boundaries
Total free space is 9871293 sectors (4.7 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1              34          614433   300.0 MiB   8300  Linux filesystem

Command (? for help):

```
You can now writes changes with the command ``w``
```
Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): Y

```
We can check again with the command ``fdisk -l`` 

```
Disk /dev/sdb: 5 GiB, 5368709120 bytes, 10485760 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 5C5371AA-C7AB-4893-93C3-EC4F18EF5839

Device     Start    End Sectors  Size Type
/dev/sdb1     34 614433  614400  300M Linux filesystem

```

Now we can see a new partition named ``/dev/sdb1``

## Install the file system 

To install the file system we will use the command ``mkfs.ext4``. This is command can be used for a ``ext4`` file system. If you need another file system, refer to the ``mkfs`` list of command. 

```
mkfs         mkfs.btrfs   mkfs.ext2    mkfs.ext4    mkfs.minix   mkfs.ntfs    mkfs.xfs
mkfs.bfs     mkfs.cramfs  mkfs.ext3    mkfs.fat     mkfs.msdos   mkfs.vfat
```

let's add the file system : 

```
mkfs.ext4 /dev/sdb1
```
The result is :

```
root@ubuntu:~# mkfs.ext4 /dev/sdb1
mke2fs 1.44.1 (24-Mar-2018)
Creating filesystem with 307200 1k blocks and 76912 inodes
Filesystem UUID: c3ff0afc-db8f-4c33-bb4c-f3b59c578b34
Superblock backups stored on blocks:
        8193, 24577, 40961, 57345, 73729, 204801, 221185

Allocating group tables: done
Writing inode tables: done
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done

```
All journal are created, we can now mount this volume in our system. 

## Mount the volume : 

There is two way to mount a volume : 

* Dynamically with the command ``mount`` 
* At the boot of the machine with the file ``/etc/fstab`` 

### The mount command 

To mount dynamically a partition to a folder we can simply use the command mount : 

```
mount /dev/sdb1 /data
```

We can check this with the list of mounted volume : 

```
mount 
```
The result is : 
```
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
udev on /dev type devtmpfs (rw,nosuid,relatime,size=989564k,nr_inodes=247391,mode=755)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
tmpfs on /run type tmpfs (rw,nosuid,noexec,relatime,size=204128k,mode=755)
/dev/sda2 on / type ext4 (rw,relatime,data=ordered)

...

/dev/sdb1 on /data type ext4 (rw,relatime,data=ordered)
```
At the end, we can see that our partition has beend mounted to the data folder with the ext4 file system and (rw,relatime,data=ordered) options. 

When you mount a volume you can specify options, for example here ``noexec`` to deny program execution : 

```
mount /dev/sdb1 /data -o noexec
```
You can find the list of available options here : http://www.tutorialspoint.com/unix_commands/mount.htm

If you reboot the server now, it won't mount automatically the partition to the folder. 

To do this we need to update the ``/etc/fstab`` file

### The fstab file 

The fstab file gathers all mounted partition in the server. 
Let's take a look to the ``fstab`` file : 
```
root@ubuntu:~# cat /etc/fstab
UUID=45403f36-cf82-4b5d-884e-4b4f46ae9ee0 / ext4 defaults 0 0
/swap.img       none    swap    sw      0       0
```
We can see that the partition with ``UUID=45403f36-cf82-4b5d-884e-4b4f46ae9ee0`` is mounted at the root ``/``

This UUID is a unique identifer of a partitions, we can list all of them with the command ``blkid``: 

```
root@ubuntu:~# blkid
/dev/sda2: UUID="45403f36-cf82-4b5d-884e-4b4f46ae9ee0" TYPE="ext4" PARTUUID="d7d1a1f9-4936-491a-addd-1cb91e7f37f9"
/dev/loop0: TYPE="squashfs"
/dev/loop1: TYPE="squashfs"
/dev/loop2: TYPE="squashfs"
/dev/sda1: PARTUUID="94ae3f1b-b7ca-458c-93db-2fb575430c26"
/dev/sdb1: UUID="c3ff0afc-db8f-4c33-bb4c-f3b59c578b34" TYPE="ext4" PARTLABEL="Linux filesystem" PARTUUID="00c9bb6a-254a-4f0e-8b8e-6a0e77a289de"
```

As we can see the partition was ``/dev/sda2``. 

If we want to mount our partition at the boot of the server we add a line in the ``fstab`` file : 

```
UUID=45403f36-cf82-4b5d-884e-4b4f46ae9ee0 / ext4 defaults 0 0
/swap.img       none    swap    sw      0       0
/dev/sdb1 /data ext4 noexec,rw
```
Save this file and use the command ``mount -a`` to automatically mount all partitions listed in this file. 

That's it ! Your disk is ready to use ! 