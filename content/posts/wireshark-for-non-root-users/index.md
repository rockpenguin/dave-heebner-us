---
title: "Wireshark for non-root users"
date: 2012-02-10
categories: 
  - "recipes"
---

Nothing Earth-shattering here, but this one always trips me up and I spend 10-15 minutes trying to remember how to set it up so that I can wireshark as a non-root user.  Works like a charm under Ubuntu, Xubuntu, and Debian 6.x.  Prolly works under all Deb derivatives, but I won't stake my career on that :-)

Anyhoo, here is the code:

sudo setcap cap\_net\_raw=+ep /usr/bin/dumpcap

Sweet! Props to the folks on the Ubuntu forums: [http://ubuntuforums.org/showthread.php?t=1086288](http://ubuntuforums.org/showthread.php?t=1086288)
