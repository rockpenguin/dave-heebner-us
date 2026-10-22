---
title: "Resizing GPT / XFS Partitions on CentOS 7 (sans LVM)"
date: 2017-04-17
categories: 
  - "recipes"
tags: 
  - "linux"
  - "storage"
---

I've resized many a ext4 partition, but now that XFS is the 'default' partition type for CentOS/RHEL 7 I am learning on the job, as the saying goes.

<!--more-->I needed to increase the partition size on a Xen VM (backed by iSCSI). So resizing the LVM virtual disk was easy-peasy. After resizing the virtual disk on the iSCSI NAS, a `parted -l` inside the VM guest showed that the disk was larger (200GB), but the partition size was still the original size (40GB). So I issued `xfs_growfs` but no change, no joy.

Turns out, [I had to delete the partition](https://serverfault.com/a/560735). So I backed up the data on the partition and fired up parted:

```
[root@backup-server ~]# parted /dev/xvdb
GNU Parted 3.1
Using /dev/xvdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) print
Model: Xen Virtual Block Device (xvd)
Disk /dev/xvdb: 215GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number Start End Size File system Name Flags
 1 1049kB 42.9GB 42.9GB xfs backups

(parted) rm 1
(parted) mkpart primary xfs 0% 100%
(parted) print
Model: Xen Virtual Block Device (xvd)
Disk /dev/xvdb: 215GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number Start End Size File system Name Flags
 1 1049kB 215GB 215GB xfs primary

(parted) quit
```

Then I issued a `partprobe` just to be on the safe side. After this I tried xfs\_growfs again:

```
[root@backup-server ~]# xfs_growfs /backups
meta-data=/dev/xvdb1 isize=512 agcount=4, agsize=2621312 blks
 = sectsz=512 attr=2, projid32bit=1
 = crc=1 finobt=0 spinodes=0
data = bsize=4096 blocks=10485248, imaxpct=25
 = sunit=0 swidth=0 blks
naming =version 2 bsize=4096 ascii-ci=0 ftype=1
log =internal bsize=4096 blocks=5119, version=2
 = sectsz=512 sunit=0 blks, lazy-count=1
realtime =none extsz=4096 blocks=0, rtextents=0
data blocks changed from 10485248 to 52428288

[root@backup-01 ~]# df -h
Filesystem Size Used Avail Use% Mounted on
...
/dev/xvdb1 200G 7.7G 193G 4% /backups
```

Viola!  And all of my data was still on the partition.
