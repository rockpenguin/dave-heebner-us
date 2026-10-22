---
title: "Joining Ubuntu SAMBA to MS Active Directory"
date: 2008-02-03
categories: 
  - "recipes"
---

Well, every time I install SAMBA I go through fits trying to remember how to link it up with Windows AD. Well, now that I've got this handy blog I am going to record it once and for all. I take no credit for any of the steps below. They are all taken from various sources I found on the 'net. YMMV...<!--more-->

1. Install SAMBA, which on Ubuntu usually seems to include winbind, kerberos, and AD support
2. Install the Kerberos tools:
    
    ```
     aptitude install krb5-user
    ```
    
3. edit /etc/krb5.conf:
    
    ```
     [libdefaults]
    default_realm = IN.DOMAIN.US
    # The following krb5.conf variables are only for MIT Kerberos.
    krb4_config = /etc/krb.conf
    krb4_realms = /etc/krb.realms
    kdc_timesync = 1
    ccache_type = 4
    forwardable = true
    proxiable = true
    v4_instance_resolve = false
    v4_name_convert =
     {  host =
       {  rcmd = host 
          ftp = ftp 
     }  plain =
       {  something = something-else  }
     }
    fcc-mit-ticketflags = true
    
    [realms]
    IN.DOMAIN.US = {  kdc = skua.in.domain.us  admin_server = skua.in.domain.us  }
    
    [domain_realm]
    .in.heebner.us = IN.DOMAIN.US
    in.heebner.us = IN.DOMAIN.US
    
    [login]
    krb4_convert = true
    krb4_get_tickets = false
    ```
    
4. Test out kerberos
    
    ```
     # kinit administrator@DOMAIN.NET
    ```
    
5. check the Kerberos keys:
    
    ```
     # klist
    ```
    
6. edit /etc/samba/smb.conf:
    
    ```
    [global]
    server string = My place on the network...
    security = ADS
    syslog = 0
    log file = /var/log/samba/log.%m
    max log size = 1000
    preferred master = No
    dns proxy = No
    panic action = /usr/share/samba/panic-action %d
    idmap uid = 5000-6000
    idmap gid = 5000-6000
    winbind separator = +
    winbind enum users = yes
    winbind enum groups = yes
    invalid users = root
    ```
    
7. edit /etc/nsswitch.conf:
    
    ```
    passwd:     compat winbind
    group:      compat winbind
    shadow:     compat
    hosts:      files dns wins
    networks:   files dns
    protocols:  db files
    services:   db files
    ethers:     db files
    rpc:        db files
    netgroup:   nis
    ```
    
8. now we join the machine to the AD domain:
    
    ```
    net ads join -U domainadminuser@DOMAIN.INTERNAL
    ```
    
9. Issue the following commands to test that we're getting the AD users & groups:
    
    ```
    wbinfo -u (or -g)
    getent passwd
    ```
