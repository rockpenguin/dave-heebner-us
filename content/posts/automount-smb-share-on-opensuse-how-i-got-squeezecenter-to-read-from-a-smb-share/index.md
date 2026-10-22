---
title: "Automount SMB Share on OpenSUSE: How I got SqueezeCenter to read from a SMB share"
date: 2008-11-02
categories: 
  - "techsupport"
---

Here are the steps:

<!--more-->

\- create the folder which you will use to mount the SMB share, e.g.:

> mkdir /media/share

\- edit /etc/fstab:

> //192.168.1.10/smbshare /media/share cifs username=user,pass=sEcrEt 0 0

\- in my case, the share wouldn't automount so I had to create a startup script to mount the drive (/etc/init.d/mount.smb):

> #!/bin/sh # ### BEGIN INIT INFO # Provides: mount.smb # Required-Start: $network # Required-Stop: squeezecenter # Default-Start: 2 3 5 # Default-Stop: 0 1 2 6 # Description: mount the SMB share ### END INIT INFO
> 
> case "$1" in start) echo -n "Mounting SMB share " mount /media/maggie\_media ;; stop) echo -n "No need to unmount share " ;; \*) echo "Usage: $0 {start|stop}" exit 1 ;; esac rc\_exit

\- Now we must make the init script part of the bootup process:

> insserv /etc/init.d/mount.smb
> 
> ln -s /etc/init.d/mount.smb /sbin/rcmount.smb

That's it - reboot to test!
