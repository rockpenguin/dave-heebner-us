---
title: "Configuring TACACS+ on Juniper SRX and Windows Active Directory"
date: 2015-04-27
categories: 
  - "recipes"
tags: 
  - "juniper"
  - "networking"
  - "windows"
draft: true
---

I'm sure someone has already documented this somewhere, but here are my usual breadcrumbs.  After pouring through Juniper's thorough, yet scattered, documentation I finally got my SRX talking to Windows Ad via TACACS+.

I decided to go with [TACACS.net](http://www.tacacs.net/), a free (not as in beer, though) command line oriented service that runs on Windows. It's a very nice program and really cool that it can be downloaded for free. They charge for support, so I guess that's how they keep the lights on.<!--more-->

1. Install TACACS.net on a domain controller and configure the software using the XML config files per the docs on the TACACS.net website.  The three main config files are:
    
    ```
    <?xml version="1.0" encoding="utf-8"?>
    <!-- Version 1.2 -->
    <Authentication xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
      <UserGroups>
        <UserGroup>
          <Name>network-admins</Name>
          <AuthenticationType>Windows_Domain</AuthenticationType>   
          <LDAPServer>127.0.0.1:389</LDAPServer>
          <LDAPUserDirectorySubtree>OU=InfoTech,dc=example,dc=com</LDAPUserDirectorySubtree>
          <LDAPGroupName>IT</LDAPGroupName>
          <LDAPAccessUserName>binduser</LDAPAccessUserName>
          <LDAPAccessUserPassword ClearText="monkey" DES=""></LDAPAccessUserPassword>
         </UserGroup>
      </UserGroups>
    </Authentication>
    ```
    
    ```
    <?xml version="1.0" encoding="utf-8"?>
    <!-- Version 1.2 -->
    <Authorizations xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
      <Authorizations>
        <Authorization>
          <UserGroups>
            <UserGroup>network-admins</UserGroup>
          </UserGroups>
          <ClientGroups>
            <ClientGroup>CoreRouters</ClientGroup>
          </ClientGroups>
          <Shell>
            <Permit>.*show.*</Permit> <!--This will allow all show commands -->
            <Deny>.*</Deny>           <!--This will deny all other commands -->
          </Shell>
          <Services>
            <Service>
              <Set>service=junos-exec</Set>
              <Set>local-user-name=network-admins</Set> <!--This is a template user on the SRX -->
            </Service>
          </Services>
        </Authorization>
      </Authorizations>
    </Authorizations>
    ```
    
    While the [Juniper documentation](http://www.juniper.net/techpubs/en_US/junos14.1/topics/task/configuration/tacacs-authentication-configuring.html) was helpful in getting started, see [Juniper KB17269](http://kb.juniper.net/InfoCenter/index?page=content&id=KB17269) for much more lucid specifics on setting up TACACS+.  Also of note is the <Services> section of this file which allows you to specify [vendor-specific attributes (VSA)](http://www.juniper.net/techpubs/en_US/junos14.1/topics/reference/general/tacacs-vendor-specific-attributes-juniper-networks.html), which is crucial for getting this to work with the SRX.
    
    ```
    <?xml version="1.0" encoding="UTF-8"?>
    <!-- Version 1.2 -->
    <!--This is the configuration file for TACACS+ clients. A TACACS+ client, as defined by the RFC, is the client that is
     making a request to the TACACS+ server such as a router, switch, or firewall-->
    <Clients xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
      <ClientGroups>
        <ClientGroup Name="CoreRouters">
          <Secret ClearText="shh-dont-tell" DES=""> </Secret>
          <Clients>
            <Client>10.6.10.1</Client>
          </Clients>
        </ClientGroup>
        <ClientGroup NAME="LOCALHOST"> <!-- This ClientGroup is just for testing -->
          <Secret ClearText="monkey" DES=""> </Secret>
          <Clients>
            <Client>10.6.15.49</Client>
          </Clients>
        </ClientGroup>
      </ClientGroups>
    </Clients>
    ```
    
    After getting everything working, you can use a handy-dandy cli tool TACDES to create DES encrypted password hashes that you can store in the file instead of the plain-text ones.
2. Now we need to setup TACACS+ on the SRX with just a few configuration lines:
    
    ```
    set system authentication-order [ password tacplus ]
    set system tacplus-server 10.6.15.49 secret ssh-dont-tell
    set system tacplus-server source-address 10.6.10.1
    set system login user network-admins class super-user
    ```
    

And that's pretty much it for basic TACACS+ setup so that you can login to your SRX with your AD credentials.
