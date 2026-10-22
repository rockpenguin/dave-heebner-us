---
title: "Resizing my LVM based virtual disk"
date: 2014-02-22
categories: 
  - "recipes"
tags: 
  - "kvm"
  - "linux"
  - "lvm"
  - "virtualization"
---

So I needed to upgrade my mail server but realized I only had 5GB of space left on the /opt partition and the upgrade complained about needing more than 5GB.  Not sure why I didn't size the whole virtual disk a little bigger in the first place.  Also not sure why I didn't set Zabbix to warn me when the disk space got that low.  Hindsight and all of that.  So following other's recipes this is how I resized my LVM based virtual disk, and then subsequently resized the partitions within the VM.

<!--more-->

OK, first task - logon to the KVM host and resize the LV that sits on my iSCSI share.  Let's see how much space/extents we have:

> root@vmsvr-2:~# vgdisplay   --- Volume group ---   VG Name               VGnas-1   System ID                Format                lvm2   Metadata Areas        1   Metadata Sequence No  16   VG Access             read/write   VG Status             resizable   MAX LV                0   Cur LV                6   Open LV               2   Max PV                0   Cur PV                1   Act PV                1   VG Size               650.00 GiB   PE Size               4.00 MiB   Total PE              166399   Alloc PE / Size       58100 / 226.95 GiB   Free  PE / Size       108299 / 423.04 GiB

Excellent! Looks like we've got a boatload of free space.  In theory you should be able to resize the guest's LV while it's still running, but in my case I shutdown the guest VMs (still working through issues of migrating guests to my other KVM host).  Not taking any chances with my mail server. With the guest VM shutdown it's just a matter of resizing the LV on the KVM host box:

> root@vmsvr-2:~# lvresize -L +30GB /dev/VGnas-1/lv-guestvm

OK, that did the trick.  We now have another 30GB to work with.  Time to reboot the VM and resize the LVs within the guest.  That of course was a little more involved, but still easy enough and can be done while the host is up and running!

The next step was to create a new LVM partition on the virtual disk.  From everything I've read so far, it seems this is the safest way to do it rather than resizing the existing PV partition.  In my case, I currently had the following allocated on the VM:

> root@mailserver:~# fdisk -l
> 
> Disk /dev/vda: 74.2 GB, 74155294720 bytes 16 heads, 63 sectors/track, 143685 cylinders, total 144834560 sectors Units = sectors of 1 \* 512 = 512 bytes Sector size (logical/physical): 512 bytes / 512 bytes I/O size (minimum/optimal): 512 bytes / 512 bytes Disk identifier: 0x000dd393
> 
> Device Boot      Start         End      Blocks   Id  System /dev/vda1   \*        2048     1953791      975872   83  Linux /dev/vda2         1953792    81917951    39982080   8e  Linux LVM
> 
> root@mailserver:~# pvscan PV /dev/vda2   VG VGlocal   lvm2 \[38.13 GiB / 0 free\]

"0 free" says it all - nothin', nada.  So I then fired up "cfdisk /dev/vda" and created a new third partition of type 8E using the empty space from the previous resize of the underlying LV.  The next step is to tell the running OS that there is another partition in town:

> root@mailserver:~# partprobe -s

which if all goes well should give you some output about the partitions it found and their types (e.g. msdos, etc.).  And another "fdisk -l" shows that our new partition is there and waiting:

> root@mailserver:~# fdisk -l
> 
> Disk /dev/vda: 74.2 GB, 74155294720 bytes 16 heads, 63 sectors/track, 143685 cylinders, total 144834560 sectors Units = sectors of 1 \* 512 = 512 bytes Sector size (logical/physical): 512 bytes / 512 bytes I/O size (minimum/optimal): 512 bytes / 512 bytes Disk identifier: 0x000dd393
> 
>    Device Boot      Start         End      Blocks   Id  System /dev/vda1   \*        2048     1953791      975872   83  Linux /dev/vda2         1953792    81917951    39982080   8e  Linux LVM /dev/vda3        81917952   144834559    31458304   8e  Linux LVM

Now that the OS knows about the new partition, we can use vgextend to extend our VG to the new /dev/vda3 partition, thereby increasing the overall size of our VG:

> root@mailserver:~# vgextend VGlocal /dev/vda3 vgextend -- INFO: maximum logical volume size is 68.13 GiB vgextend -- volume group "VGlocal" successfully extended

Now if we issue a "vgdisplay" we should see that we have plenty of extra space/extents to now increase the size of the LVs.  So let's go ahead and resize our LV where we have the /opt partition mounted:

> root@mailserver:~# lvresize -L +15GB /dev/VGlocal/opt

In this case I only increased it by 15GB so that I'll have another 15GB to play with if I need to increase the root or swap partitions.  Almost there.  Now we have to resize the filesystem that sits atop of the resized LV:

> root@mailserver:~# resize2fs /dev/VGlocal/opt resize2fs 1.42 (29-Nov-2011) Filesystem at /dev/VGlocal/opt is mounted on /opt; on-line resizing required Resizing the filesystem on /dev/VGlocal/opt to 8814592 blocks. The filesystem on /dev/VGlocal/opt is now 8814592 blocks long.

and if we take a peek at /var/log/syslog we see:

> Feb 22 12:52:13 mailserver kernel: \[ 4692.826283\] EXT4-fs (dm-1): resizing filesystem from 4882432 to 8814592 blocks Feb 22 12:52:14 mailserver kernel: \[ 4693.606755\] EXT4-fs (dm-1): resized filesystem to 8814592

As I'm just using up all of the new space on the resized LV, no arguments for resize2fs are necessary.  So now if we issue another "df -h" then we should see the newly resized /opt partition!

> root@box106:~# df -h Filesystem                Size  Used Avail Use% Mounted on /dev/mapper/VGlocal-root   15G  3.9G  9.9G  29% / udev                      2.0G  4.0K  2.0G   1% /dev tmpfs                     791M  244K  791M   1% /run none                      5.0M     0  5.0M   0% /run/lock none                      2.0G     0  2.0G   0% /run/shm /dev/vda1                 922M  133M  743M  16% /boot /dev/mapper/VGlocal-opt    33G   12G   20G  38% /opt

Success, and with minimal downtime to my mail server.  Now we can proceed with the upgrade...

* * *

Thanks to the Gentoo wiki, always very thorough with their documentation: [http://wiki.gentoo.org/wiki/LVM](http://wiki.gentoo.org/wiki/LVM), as well as the article over at [http://blog.pluralsight.com/resize-vmware-linux-drives](http://blog.pluralsight.com/resize-vmware-linux-drives)
