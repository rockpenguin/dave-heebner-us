---
title: "XenServer 7, Boot Errors, FreeNAS, and MPIO"
categories: 
  - "recipes"
draft: true
---

![xenserver\_logo](images/xenserver_logo.png)It was time to update my XenServer lab environment.  I've been running 6.5 and everything has been running great. But 7 came out with some long-awaited features (>4GB root partition, UEFI support) so time to upgrade as well as simplify my XenServer setup.<!--more-->

The sane partitioning and working UEFI are great.  Back on 6 I had a host go South due to a full root partition caused by logs and and leftover patch files (luckily there was [this workaround](https://major.io/2012/01/13/xenserver-6-disable-gpt-and-get-a-larger-root-partition/)). Normally I don't care much about UEFI, but the Lenovo ThinkCentre machines I use for my XenServer hosts suffer from [this](http://discussions.citrix.com/topic/330265-error-1962-no-operating-system-found-after-successful-install-on-thinkcentre-m82/) "[bug](https://ulrikchristensen.com/xenserver-will-not-boot-after-installation/)" so every time I would reinstall XenServer I get the dreaded "_Error 1962: No operating system found_" and no matter what I set the BIOS boot settings to it doesn't work due to the GPT partitions.  Anyhoo, fast forward to XenServer 7 and the UEFI support now works like a champ and the install process goes smooth!

But the real reason for the upgrade was a simplification of my setup. In the initial setup of my XenServer/iSCSI environment I had over-engineered the storage network so that I could use iSCSI and MPIO:

- Two (2) Lenovo ThinkCentre Xeon-based workstations
- Two (2) HP 2920-24G switches with stacking modules and cables
- FreeNAS Mini providing iSCSI
