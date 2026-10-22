---
title: "Removing a Virtual Machine from VirtualBox"
date: 2008-03-16
categories: 
  - "recipes"
tags: 
  - "virtualbox"
---

Below are the steps to remove a virtual machine from the command line:

1. Remove the disk from the VM: VBoxManage modifyvm xfce-test -hda none
2. Unregister the VM: VBoxManage unregistervm xfce-test
3. Unregister the disk (vdi): VBoxManage unregisterimage disk /path/to/vdi
