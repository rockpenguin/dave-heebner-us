---
title: "K.Y.V.S.S."
date: 2017-05-14
categories: 
  - "reviews"
  - "rants"
tags: 
  - "kvm"
  - "virtualization"
  - "xen"
---

![XenPanda](images/xenpanda.jpg)I have a a disorder when it comes to my home lab. I always have to try something else, whether it's the latest technology or just something "I've wanted to try". This leads to a lot of reinstalling servers and networks. This is not all bad as I learn best by _doing_.  This time I wanted to revamp my lab virtualization.<!--more-->

I use XenServer for my virtualization and I love it. It's open-source, rock solid and easy to setup and use.  The downside is that the Xen API is goofy (XML-RPC) and very little of the IaaS tooling utilizes it. So no Terraform, no Vagrant, not even libcloud. Granted these tools _do_ support libvirt, which supports Xen, but not XenServer. I guess this is a holdover from Citrix doing something proprietary. Anyhoo.

So I figured I'd give KVM another shot.  I need Windows support (unfortunately), so no LXC/LXD, Docker, BSD Jails, and besides, [this](https://abe-winter.github.io/blues/2017/04/27/config-vs-containers.html) which I agree with. I started out with KVM when I ventured into virtualization back in 2006, but management tooling sucked and it stunk compared to VMware.  A recent client I worked with used KVM for their virtualization and I had flashbacks of the cobbled together scripts and tooling and trying to remember the God awful CLI steps just to resize a vdisk. I'm sure if you work with it everyday, then it's great but I just want my virtualization to work without having to spend 20 minutes searching for the required argument to qemu.

So I've been eyeing up oVirt for quite some time and figured I'd take the plunge. I setup the management machine (oVirt Engine), using a separate machine just for that purpose. Interface was slick, everything was looking good. I setup my networking, then my FreeNAS iSCSI shared storage. Then I tried to setup a node and things started going south. The install ISO oVirt provides installed without any issues, but I could never get the oVirt Engine app to connect to the node. Ugh. So I then manually installed CentOS 7.2 on one of the node boxes (Lenovo Xeon ThinkCentres) and followed the instructions for setting up KVM manually. Still no go. I did finally get it to work, and once I did it worked fairly well, with only a few hiccups with the networking. This is in no way a reflection on the oVirt team and it probably had something to do with my setup. It's a really cool product with a lot of work that has gone into it.  However, my philosophy has always been, if it is difficult to setup, then what happens 6 months later when it fails and I have to remember how to put Mr. Humpty back together again. Granted, some of the best learning happens during failure, but if failure or over complication can be avoided in the first place then that's the path I'll choose.

Long story short, I stopped spending time on KVM and reinstalled XenServer on the nodes and that's where I am staying.  Lesson learned, if it ain't broke leave it alone. KYVSS - Keep your virtualization simple stupid.
