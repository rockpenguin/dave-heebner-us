---
title: "Ubuntu and mount.cifs"
date: 2008-05-21
categories: 
  - "recipes"
  - "techsupport"
tags: 
  - "linux"
  - "samba"
---

Using SMB/CIFS under Nautilus was too slow. For Windows shares that I use regularly I figured it would be faster and easier to have the share mounted permanently as part of the file system. This also has the benefit of all Linux programs being able to access the files, not just Gnome based apps. Here is how I mounted a R/W Windows share under Linux.<!--more-->

My experience with Samba is that getting it to work properly varies from installation to installation, so your mileage may vary. More information on CIFS under Linux can be found here: [http://linux-cifs.samba.org/](http://linux-cifs.samba.org/)

Here are the vitals:

- Ubuntu 7.10
- Samba related packages that are installed: smbclient, libsmbclient, smbfs, samba-common
- /etc/samba/smb.conf: workgroup = MYDOMAIN wins server = 192.168.1.10

First, let's install "smbfs" (if it's not already installed):

> sudo aptitude install smbfs

Next we need to create the folder on the local file system that mount.cifs will use. With Ubuntu, you can create the folder where mount puts things like CD-ROMs and USB flash drives (/media), and using the same name as your Windows server share:

> sudo mdkir /media/sharename

You will also want to give this folder permissions so that regular users can read/write to the folder:

> sudo chmod 777 /media/sharename

or if you want to be a little more paranoid:

> sudo chown root.somegroup /media/sharename
> 
> sudo chmod 770 /media/sharename

where "somegroup" is a group that contains the users who needs access to this share. This is partly what will allow you to access the mounted share as R/W. Running the command " ls -l /media" should show your permissions as follows:

> drwxrwx--- 2 root somegroup 4096 2008-05-21 09:04 sharename

OK, now we have to pass the credentials to the SMB/CIFS server but we don't want to put the username/password into the fstab file, so we will create a file containing this info. A good place to put it might be under "/root":

> sudo nano /root/.smbcreds

and add the following lines

> username=johndoe password=mysecret

Now we edit /etc/fstab and add the following line at the end:

> //servername/sharename /media/share cifs credentials=/root/.smbcreds,rw,iocharset=utf8,setuids,file\_mode=0666,dir\_mode=0777 0 0

If you have a share that has spaces in the name, e.g. "serverour docs" then you will need to replace the space with `{backslash}040` so that your line in /etc/fstab should look something like:

> `//servername/our{backslash}040docs ...`

NOTE: I had to replace the actual backslash with {backslash} because WordPress kept removing the backslash and zero.

Finally, go ahead and mount the drive:

> sudo mount -a
