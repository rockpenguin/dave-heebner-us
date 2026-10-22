---
title: "Wireless Mesh (almost there...)"
date: 2011-12-10
categories: 
  - "reviewsrants"
tags: 
  - "cloud"
  - "networking"
  - "open-source"
  - "wireless"
coverImage: "om1p_top.jpg"
---

## Overview

I've been testing a wireless mesh system for the past few weeks and thought I'd post my thoughts.  The system I've been testing is the [Open-Mesh](http://www.open-mesh.com/), and more specifically the [OM1P](http://www.open-mesh.com/index.php/professional/professional-mini-router-us-plugs.html) which is a pretty slick little device.  For those unfamiliar with the Open-Mesh products, it is designed to be a plug-n-play "public" wi-fi setup, but with an extra twist - these things all talk to one another and create a wireless mesh which allows anyone to quickly setup wi-fi hotspots.  <!--more-->From the Open-Mesh website:

> Open-Mesh creates ultra low-cost zero-config, plug & play wireless mesh network solutions that spread an Internet connection throughout a hotel, apartment, office, neighborhood, village, coffee shop, shopping mall, campground, marina and just about anywhere else you can imagine.
> 
> We provide all the components you need: access points, tool-free indoor wall-plug and outdoor enclosures for simple installations, and our free cloud-based controller. Set the bandwidth users get, design your own splash pages, track users, & monitor your network's health using just a browser from anywhere in the world.

It's a pretty slick system and really doesn't require a guru to setup, although there is still a wee bit o' technical knowledge required as the system still uses terms such as SSID, access point, etc.  But Open-Mesh has some good explanations on their website.

\[caption id="attachment\_93" align="alignright" width="300" caption="OM1P ports"\][![OM1P ports](images/om1p_side_and_rear-300x141.jpg "om1p_side_and_rear")](http://www.rockpenguin.com/wp-content/uploads/2011/12/om1p_side_and_rear.jpg)\[/caption\]

##  How Does It Work?

The system is designed to be cloud-based, so it's a little backwards from the traditional access point setup.  So you don't need to gain access to the device to setup the wi-fi network - no setting your IP address to a static one, no fiddling with DHCP.

- The access points are called **nodes**. Basically, you plug the first node into your wired network, or directly into your Internet router/firewall if you don't have a wired network. This will be the "gateway" node.  If you are using multiple nodes, you can just plug the rest into power outlets around your home, coffee shop, etc. (the OM1P does work with PoE, but it does not come up all of the way with my older [802.3af](http://en.wikipedia.org/wiki/PoE) 15w switch)
- Then head over to [Open-Mesh](http://www.open-mesh.com/index.php/) and click on the link labeled **Dashboard**. This will take you to a site called [**CloudTrax**](https://www.cloudtrax.com/dashboard.php) and will allow you to setup all of the juicy details of your wireless network.
- Once you create an account, you can add nodes to your account by placing them on Google maps and entering the MAC address of the nodes. The map feature allows you to locate your nodes where they are actually placed geographically.
- In the CloudTrax portal you can set the wireless particulars such as SSID#1 (public) and SSID#2 (private), WPA passwords, etc. You can also create a custom splash page that SSID#1 users will be redirected to when they first try to access the 'net.

That's the setup in a nutshell. There are a few more bells and whistles that you can configure, but I won't bore you with any more details.  For more instructions, the Open-Mesh folks have a [good tutorial chock full o' screenshots](http://support.open-mesh.com/knowledgebase/10/Using-the-Cloud-Controller-Dashboard.html).

## What I Found

-  The mesh capabilities work well, although you have to keep the mesh hop count low as the bandwidth decreases with each node that is not connected directly to the network; the documentation recommends that you place the gateway node close to the center of your mesh
- SSID#1 is separated from the local LAN, making it a "guest" wi-fi network
- SSID#2 can be bridged with the local LAN, but this functionality requires a beta firmware called Firmware NG
- The Firmware NG can be applied via the CloudTrax portal, but this never worked for me so I [manually flashed](http://support.open-mesh.com/knowledgebase/43/How-to-Manually-Flash-NG-on-a-cloudtrax-enabled-router.html) my two OM1P nodes
- There were some issues with the wireless LAN dropping, but I think I might have had the two nodes too close together.  I moved them further apart and that seems to have helped
- The devices only use WPA for both SSID#1 and SSID#2, so I wouldn't want to use these in a setting where security could be an issue; i.e. connecting these to a network in a coffee shop that was connected to POS or credit card terminals
- One little nagging issue for me is the security - the open-mesh.com website doesn't really have any specifics about security used for inter-node communication, or communication with the CloudTrax portal for that matter
- Another scary thought is... what if the CloudTrax website is hacked? In theory the hackers have every piece of information about your wireless network - where it is located, what the WPA keys are, as well as the root password to SSH into the devices; granted, you don't have to use the map feature (you could put all of your nodes anywhere on the map), but it is still a little worrisome for me

Overall these are great devices at a great price and would be perfect for many types of wireless roll-outs such as hotels, apartment buildings, and even smaller installations such as cafes and coffee shops.  I would like to see more about the security of the devices, but I suppose I could just break out the sniffer and analyze the traffic myself.  Maybe for another post :-)
