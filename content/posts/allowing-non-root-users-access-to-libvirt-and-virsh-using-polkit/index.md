---
title: "Allowing non-root users access to libvirt and virsh using polkit"
date: 2014-03-09
categories: 
  - "recipes"
---

I've been using [virt-manager](http://virt-manager.org/) to manage my KVM hosts and I'm not keen on having to login to the remote hosts as root, plus I would get the password prompt every time I connect to the server (sure I could setup my pulic SSH key on the root account, but not a good idea to use RSA auth to the root account on a remote server).  With Debian (Wheezy) it was fairly simple in that all that I had to do was add my regular username to the group "libvirt".  Then I could use the URI: qemu+ssh://virtadmin@my.kvmhost.com/system to connect to the remote KVM host using virt-manager.

<!--more-->

I still had trouble doing things like migration and I couldn't use virsh from the CLI.  Not the the end of the world, just figured it was a bug or oddity because I am using Fedora as my laptop distro and Debian on the KVM host.  But I could use virt-manager to create and manage my guests remotely.

As usual, I get itchy and want to try new distros and such so I wanted to see what KVM was like under Centos 6 since that's what I use at work.  So I migrated all of my guests over to one KVM host, then installed CentOS 6.5 minimal on the other server and installed all of the necessary KVM and libvirt RPMs.

> $ yum install libvirt libvirt-client libvirt-python qemu-kvm libguestfs-tools

But I soon found out that no matter which group I added my username to under CentOS, I couldn't do anything as a normal user, either from virt-manager or the CLI on the server.  So after some Googlin' I came across the [libvirt wiki page outlining how to use PolicyKit](http://wiki.libvirt.org/page/SSHPolicyKitSetup) to allow normal users to access and use libvirt.  Cool!  And thankfully since CentOS 6.5 still uses PolKit 0.95 I don't have to create the file in JavaScript!  So I followed the instructions, added my username to the libvirt group and viola! I could now access the remote server from virt-manager.  However, I still couldn't do anything using virsh from the CLI on the host server.  Turns out you need to specify the URI when using virsh even though you are part of a group that has full access to libvirt.  Using [this post](http://www.linuxsysadmintutorials.com/configure-polkit-to-run-virsh-as-a-normal-user/), I added the necessary stanza to .bash\_profile and was off and running:

> $ vi ~/.bash\_profile
> 
> \# the following stanza will allow the use of virsh without having \# having to specify the qemu+ssh URI every time if test -x \`which virsh\`; then   export LIBVIRT\_DEFAULT\_URI=qemu:///system fi
> 
> $ source ~/.bash\_profile

SELinux Note: If you keep SELinux on (SELINUX=enforcing in /etc/selinux/config), then you will need to follow the instructions on [this post](http://thepoch.com/tumblr/so-you-want-a-centos-6-kvm-host-server.html) to make sure that libvirt can properly access resources on folders, NFS mounts, etc. that aren't part of the default locations setup when you install KVM/qemu/libvirt.  Me, I've set it to SELINUX=permissive so I don't have to constantly guess why I get errors when trying to do something. Gotta love SELinux.  Maybe someone can explain why I should keep it on...
