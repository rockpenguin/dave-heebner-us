---
title: "Debian LDAP / PDC"
categories: 
  - "techsupport"
draft: true
---

The goal: to create a Linux server that can replace a Windows 2003 PDC

Most of this was derived from:

- http://mds.mandriva.org/content/MMC/install/en/mmc-generic-installation.html
- http://www.howtoforge.com/mandriva-directory-server-on-debian-etch

1. install Debian 4.0 (Etch); I went with a minimal install, so I didn't select any tasks from tasksel, not even the last option of standard config.
2. aptitude install openssh-server
