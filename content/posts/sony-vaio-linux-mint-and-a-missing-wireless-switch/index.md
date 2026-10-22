---
title: "Sony VAIO, Linux Mint, and a (missing) wireless switch"
date: 2012-10-01
categories: 
  - "recipes"
---

I am the proud new owner of a Sony VAIO S-series laptop (model# SVS151190X), running [Linux Mint](http://www.linuxmint.com/) 13 (Maya).  Everything installed beautifully and all hardware was found.  That was a first for me and a new laptop, probably a testament to the hard work of the Linux devs.  Also very glad to be done with my MacBook Pro.  Anyhoo, I recently disabled the wireless from the network manager menu (right-click, uncheck 'Enable Wireless').  When I went to re-enable it it was grayed out and I couldn't re-enable the wireless.

<!--more-->

The Network Manager menu stated "wireless disabled by hardware switch".  The only problem with that is that this laptop doesn't have a physical switch, nor does it have a Fn+ key combo to enable/disable the wireless.  I tried 'sudo ifup wlan0' and 'sudo ifup phy0' but with no joy.  A reboot or two did nothing to bring it back, either.  A couple of Google searches lead me to this askubuntu.com forum post:

[http://askubuntu.com/questions/181108/sony-vaio-pcg-71315l-wireless-is-disabled-by-hardware-switch-help](http://askubuntu.com/questions/181108/sony-vaio-pcg-71315l-wireless-is-disabled-by-hardware-switch-help)

So I just had to issue the following command:

> **dave@vaiox ~ $** rfkill list 0: sony-wifi: Wireless LAN Soft blocked: yes Hard blocked: no 1: sony-bluetooth: Bluetooth Soft blocked: no Hard blocked: no 2: hci0: Bluetooth Soft blocked: no Hard blocked: no 3: phy0: Wireless LAN Soft blocked: yes Hard blocked: yes

Huh, never knew about this handy command.  Still learning after 10+ years of using Linux.  So I just issued the command:

> **dave@vaiox ~ $** sudo rfkill unblock all

and all was right in the world again.
