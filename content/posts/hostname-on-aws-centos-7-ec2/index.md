---
title: "Hostname on AWS CentOS 7 EC2"
date: 2016-01-17
categories: 
  - "recipes"
  - "techsupport"
tags: 
  - "aws"
  - "linux"
---

I was setting up a shiny new CentOS 7 EC2 instance, but when I tried to set the hostname using all of the typical Linux-y ways, none of them stuck after a reboot.  It just kept going back to the default EC2 naming convention of 'ip-172.31.x.x'.  Since I am still getting used to CentOS 7 and all of the stuff they changed from 6, I figured it was a CentOS 7 thing.  Not so...

<!--more-->

Turns out it was due to the EC2 instance being provisioned with [cloud-init](https://help.ubuntu.com/community/CloudInit) which AWS uses to configure the AMIs.  First I headed over to the [cloud-init docs](http://cloudinit.readthedocs.org/en/latest/index.html) to see what I could glean from RTFM.  Meh, pretty useless.  So after some searching I came across [AWS' documentation](https://aws.amazon.com/premiumsupport/knowledge-center/linux-static-hostname-rhel7-centos7/) and a few posts that outlined [some](http://serverfault.com/questions/645099/how-to-override-the-hostname-that-dhcp-configures-on-a-centos7-box-in-aws) [examples](https://ask.fedoraproject.org/en/question/37413/correctly-setting-the-hostname-fedora-20-on-amazon-ec2/) of parameters to change in the cloud-init configuration.  So for me, the config that worked was the following:

```
# echo my-host1 > /etc/hostname (or use your fav editor)
# vim /etc/cloud/cloud.cfg
```

and make sure that cloud.cfg file has the following line at the bottom (noted in red):

```
system_info:
  default_user:
    name: centos
    lock_passwd: true
    gecos: Cloud User
    groups: [wheel, adm, systemd-journal]
    sudo: ["ALL=(ALL) NOPASSWD:ALL"]
    shell: /bin/bash
  distro: rhel
  paths:
    cloud_dir: /var/lib/cloud
    templates_dir: /etc/cloud/templates
  ssh_svcname: sshd

preserve_hostname: true
```

NOTE: notice that the line "preserve\_hostname" has no indentation, e.g. "top level".  This had me flummoxed for a reboot or two.

Now when I reboot, my hostname is correctly set!

```
[centos@my-host1 ~]$ hostname
my-host1
[centos@my-host1 ~]$ hostname -f
my-host1.useast.aws.example.net
```

Perfect!  For my FQDN I use a private Route 53 zone and have my DHCP options set to use that zone as the default for my VPC so that the hosts automatically pull the domain name.
