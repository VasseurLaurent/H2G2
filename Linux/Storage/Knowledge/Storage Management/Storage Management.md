# Storage Management in a Linux OS

**Estimation reading : 10 mins**

**Date : 26/05/2019**

## GPT & MBR :books:

You probably know that you can split your hard disk into several partitions. The question is, how does the OS know the partition structure of the hard disk? That information has to come from some where. This is where MBR (Master Boot Record) and GPT (Guid Partition Table) come into play. While both are architecturally different, both play the same role in governing and provide information for the partitions in the hard disk.

### MBR
MBR is the old standard for managing the partition in the hard disk, and it is still being used extensively by many people. The MBR resides at the very beginning of the hard disk and it holds the information on how the logical partitions are organized in the storage device. In addition, the MBR also contains executable code that can scan the partitions for the active OS and load up the boot up code/procedure for the OS.

For a MBR disk, you can only have ``four primary partitions``. To create more partitions, you can set the fourth partition as the extended partition and you will be able to create more sub-partitions (or logical drives) within it. As MBR uses 32-bit to record the partition, each partition can only go up to a ``maximum of 2TB in size``. This is how a typical MBR disk layout looks like:


![mbr-disk.png](.attachments\mbr-disk-layout.png)

There are several pitfalls with MBR. First of all, you can only have 4 partitions in the hard disk and each partition is limited to only 2TB in size. This is not going to work well with hard disk of big storage space, say 100TB. Secondly, the MBR is the only place that holds the partition information. If it ever get corrupted (and yes, it can get corrupted very easily), the entire hard disk is unreadable.

### GPT 

GPT is the latest standard for laying out the partitions of a hard disk. It makes use of globally unique identifiers (GUID) to define the partition and it is part of the UEFI standard. This means that on a UEFI-based system (which is required for Windows 8 Secure Boot feature), it is a must to use GPT. With GPT, you can create theoretically unlimited partitions on the hard disk, even though it is generally restricted to 128 partitions by most OSes. Unlike MBR that limits each partition to only 2TB in size, each partition in GPT can hold up to 2^64 blocks in length (as it is using 64-bit), which is equivalent to 9.44ZB for a 512-byte block (1 ZB is 1 billion terabytes). In Microsoft Windows, that size is limited to 256TB.

![gpt-partition-scheme.png](.attachments\gpt-partition-scheme.png)


### File systems 

It exists a lot of different file systems with differents uses cases : 

* Ext2,3,4 : The ext4 journaling file system or fourth extended filesystem is a journaling file system for Linux. This filesystem is the most common file system for Linux . It provides a good system to store normal files 
* xfs : This filesystem is designed to store large file, it is the default file system for CentOS. 

### Tools to manage partitions

In this part, I will define how to setup, partition and mount a new disk in the system. 
To setup a new disk, we can use 3 tools : 

* fdisk : Old tool, it doesn't support ``GPT``
* parted : Installed on all Linux distribution, it is a good tool to manage ``GPT/MBR`` . 
* gdisk : New tool mainly installed on Ubuntu 
* df -hT : Command which list all mounted file systems

# Bibliography 

## French resources 

[MBR & GPT](https://www.tech2tech.fr/quelle-est-la-difference-entre-le-format-gpt-et-mbr-pour-un-disque/)

