---
title: "Adding a VirtualBox Guest via the Command Line"
date: 2008-02-24
categories: 
  - "recipes"
draft: true
---

Needs work...

<!--more-->

1. Install the bridge utilities: aptitude install bridge-utils
2. Edit /etc/network/interfaces
    
    ```
    # This file describes the network interfaces available on your system
    # and how to activate them. For more information, see interfaces(5).
    
    # The loopback network interface
    auto lo
        iface lo inet loopback
    
    # The primary network interface
    auto eth0
        iface eth0 inet static
        address 192.168.100.11
        netmask 255.255.255.0
        gateway 192.168.100.1
        network 192.168.100.0
        broadcast 192.168.100.255
    
    auto eth1
        iface eth1 inet static
        ifconfig eth1 0.0.0.0 up
        up ip link set eth1 promisc on
        down ip link set eth1 promisc off
        down ifconfig eth1 down
    
    auto br0
        iface br0 inet static
        bridge_ports eth1 vbox0
        address 192.168.100.12
        netmask 255.255.255.0
        gateway 192.168.100.1
    ```
    
3. Add the virtual tap interface using the VirtualBox tool
    
    ```
    VBoxAddIF vbox0 <user> br0
    ```
    
4. wget http://archive.ubuntu.com/ubuntu/dists/gutsy/main/installer-i386/current/images/netboot/mini.iso
5. mv mini.iso ubuntu\_gutsy\_7.10\_mini.iso
6. VBoxManage createvm -name testpc -register -basefolder /path/to/vm/
7. VBoxManage createvdi -filename /path/to/vm/testpc/testpc.vdi -size 5000 -register
8. VBoxManage modifyvm testpc -hda /path/to/vm/testpc/testpc.vdi
9. VBoxManage modifyvm erp -memory 256MB -nic1 hostif -hostifdev1 vbox0
10. VBoxManage modifyvm erp -vrdp on -vrdpport 3390
11. VBoxManage modifyvm erp -dvd /path/to/iso
12. VBoxManage startvm <name> -type vrdp

Removing a VM

1. First we have to detach the VDI disk from the VM: VBoxManage modifyvm <name> -hda none
2. Then we must unregister and delete the VM: VBoxManage unregistervm <name> -delete
3. Lastly, let's delete the VDI: VBoxManage unregisterimage disk /path/to/vm/disk.vdi
