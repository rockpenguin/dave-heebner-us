---
title: "Replacing (or transfering FSMO roles to) a Windows 2003 Domain Controller"
categories: 
  - "recipes"
  - "techsupport"
draft: true
---

Every time I use a Microsoft product anymore, I'm reminded why I like Linux and FOSS so much. Anyhoo, here are my notes for how to replace (and/or transfer roles to) a Windows 2003 domain controller.

Scenario: I have a Windows 2003 domain controller that I want to move to another machine (in this case a virtual server). So I just need to get all of the FSMO roles onto the new server, right?

Here are the steps I took. They may not be the right steps, but they worked for me :-) YMMV. Some of the links I used for this process were:

- [Decommissioning a Domain Controller](http://technet2.microsoft.com/windowsserver/en/library/b96e3f0a-bf98-4a80-8718-dd80dc1071fd1033.mspx?mfr=true "Decommissioning a Domain Controller")
- [Using Ntdsutil.exe to transfer or seize FSMO roles to a domain controller](http://support.microsoft.com/kb/255504)
- [How to remove data in Active Directory after an unsuccessful domain controller demotion](http://support.microsoft.com/kb/216498)

1. Installed Windows 2003 on the new server.
2. Gave the server the DNS Server role. (NOTE: The first time I did this process, I actually did steps 2-4 after the ntdsutil et. al. so that might have been my undoing. Maybe doing these role changes first might have eliminated the need for editing the DNS further down)
3. Network settings -> TCP/IP -> make primary DNS server the new server's IP
4. Gave the server the File Server role.
5. Used dcpromo to make the server a domain member (not a new domain server).
6. Used ntdsutil
7. Reboot the new server.
8. Make the new server host the Global Catalog
    - Start -> Administrative Tools -> Active Directory Sites and Services
    - Sites -> Default-First-Site-Name -> Servers -> <new server name>
    - Right-click on NTDS settings and check "Global Catalog"
9. Open the DNS server utility
    - Forward Lookup Zones -> <domain name> -> \_msdcs
    - then double-click on "Name Server (NS)" record
    - If it's not already in the list of name servers, add the new server and IP
    - Sites -> Default-First-Site-Name -> \_tcp
    - change the \_gc SRV record to the new server name
10. Start -> Administrative Tools -> Active Directory Users and Computers
     - Right-click on domain name, choose "Raise Domain Functional Level"
     - Choose Windows 2003
11. Rename domain controller:
     - Active Directory Sites and Services: Sites -> Default-First-Site-Name -> Servers; make sure the old server name is not in the list of servers - if it is, delete it.
     - netdom computername skua-new.in.heebner.us /add:skua.in.heebner.us Successfully added skua.in.heebner.us as an alternate name for the computer.
