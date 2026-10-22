---
title: "Installing pfSense on the alix2c1"
date: 2008-02-23
categories: 
  - "recipes"
  - "techsupport"
tags: 
  - "alix"
  - "embedded"
  - "firewall"
  - "geekspeak"
  - "m0n0wall"
  - "pfsense"
---

So, what is [pfSense](http://www.pfsense.org "pfSense firewall"), and more importantly what the heck is an [alix2c1](http://http://www.pcengines.ch/alix.htm "ALIX system boards")? pfSense is a complete purpose-built firewall software that can be installed on a PC, as well as embedded platforms such as [PC Engines ALIX boards](http://www.pcengines.ch/alix.htm). The focus of this recipe is to build an embedded firewall appliance. Why would I do this when a cheap firewall can be purchased off of the shelf? Simple, the cheaper firewalls don't come with very many features (VPN built in, RRD network graphs, 3rd LAN port for "orange" DMZ setups, etc.). To get these features in a commercial firewall appliance means spending in the neighborhood of $300 as well as being locked into a certain vendor for VPN, etc. Making my own costs about $150 and give me a lot of flexibility.

So, this is a story of how I spent the past two nights sitting cross-legged on the floor with a laptop connected to a tiny motherboard that would become a firewall/router.<!--more-->

The instructions that I found on both the pfSense website and the m0n0wall website were for installing on the End-of-life WRAP product, but I figured it had to be similar, right? Well, sort of :-) After following the instructions on the [pfSense website](http://www.pfsense.org/index.php?id=36) and then the [m0n0wall website](http://doc.m0n0.ch/quickstartwrap/#Setup.Installing), things started to go downhill. The device would boot, pfsense would go through its loading process, but then just die on the loading of the DHCP server. Never got an IP address on any of the NICs - but each NIC module got its link light when I plugged in the LAN cable so that was a sign that at least the NIC modules were working on some level. So I figured this is a BETA version of pfSense, so maybe I need to install m0n0wall. m0n0wall would boot up just fine and even made it to its menu, but I could never get the NICs to come up. None of them. Could I have a bit o' bad hardware? So I hit the web again and soon found out from the forums over at pfsense that a BIOS upgrade might be in order.

The following steps are how I finally got this project off of the ground. I used Linux to do my bidding, but you can certainly do this from a Windows box (some of the steps are different, though...)

Hardware you will need (I purchased from [Netgate](http://www.netgate.com/) and it came fast and furious, but there is a list of vendors on [PC Engines' website](http://www.pcengines.ch/order.php)):

- [ALIX2c1](http://www.netgate.com/product_info.php?manufacturers_id=26&products_id=502) or another embedded platform from PC Engines.
- [Aluminum enclosure](http://www.netgate.com/product_info.php?products_id=292) (optional if you're going to install your ALIX in another enclosure).
- [Power supply](http://www.netgate.com/product_info.php?products_id=357) to power the ALIX (optional if you're going to use Power over Ethernet, or PoE).
- A Compact Flash (CF) card (has to be at least 128MB according to the pfSense website).
- A Compact Flash reader connected to your PC. You'll need this to write the image to the CF card.
- A computer with a serial port, preferably one with a true serial port and not a USB-to-serial adapter. It might work just fine with the USB/serial but if it doesn't then you'll have one more thing to troubleshoot.
- A DB9 null-modem cable with a DB9 female on both ends (or some gender changers to get you there)

When you get the hardware - DO NOT mount the ALIX board into the enclosure until you get it working. You might be removing/inserting the CF card a lot and the enclosure doesn't give you any room to remove the CF card. OK, now that you've got all of the necessary hardware, here is the recipe for cooking up your own firewall:

1. [Download pfSense for embedded platform](http://www.pfsense.org/index.php?id=22). The most current version at the time of this writing was 1.2-RC4, which I found to be stable for my needs as I had been using the PC based version for several weeks.
2. Unzip the downloaded file (the version I downloaded isn't a tar archive so we only need to use gunzip):
    
    ```
    gunzip pfSense-1.2-RC4-Embedded.img.gz
    ```
    
    This will expand the file, leaving a file called pfSense-1.2-RC4-Embedded.img.
3. Now insert the CF card into the card reader on your PC. To find out the resource Linux is using to access the drive, type the following at the command line:
    
    ```
    dmesg
    ```
    
    and look at the last several lines which might look something like:
    
    ```
    [ 7377.984000] sd 2:0:0:0: [sdb] 700560 512-byte hardware sectors (359 MB)
    [ 7377.984000] sd 2:0:0:0: [sdb] Write Protect is off
    [ 7377.984000] sd 2:0:0:0: [sdb] Write cache: disabled, read cache: enabled, doesn't support DPO or FUA
    [ 7377.984000] sd 2:0:0:0: [sdb] 700560 512-byte hardware sectors (359 MB)
    [ 7377.984000] sd 2:0:0:0: [sdb] Write Protect is off
    [ 7377.984000] sd 2:0:0:0: [sdb] Mode Sense: 00 3a 00 00
    [ 7377.984000] sd 2:0:0:0: [sdb] Write cache: disabled, read cache: enabled, doesn't support DPO or FUA
    [ 7377.984000]  sdb: sdb1
    [ 7377.984000] sd 2:0:0:0: [sdb] Mode Sense: 00 3a 00 00
    [ 7378.020000] sd 2:0:0:0: [sdb] Attached SCSI removable dis
    [ 7378.020000] sd 2:0:0:0: Attached scsi generic sg2 type 0
    ```
    
    From this output we can see that Linux is accessing the CF card as /dev/sdb.
4. Next we will use the Linux utility "dd" to write the image to the CF card
    
    ```
    dd if=pfSense-1.2-RC4-Embedded.img of=/dev/sdb
    ```
    
    where "if=" is the input file (the pfSense image) and "of=" is for the output file. In our case we are writing the output to the file that is the CF card (remember, in [Linux almost everything is a file](http://www.linux.org/lessons/beginner/l4/lesson4e.html)). While the process is running, unfortunately you won't get any output or indication of what it is doing. However, when it is finished you should get some output such as:
    
    ```
    239144+0 records in
    239144+0 records out
    122441728 bytes (122 MB) copied, 187.177 seconds, 654 kB/s
    ```
    
5. Unmount the CF card from your computer. If you've got a nifty neato X session going, and a handy dandy automounter, then chances are good that you can just right-click on the respective desktop icon for your CF card and select "Unmount Volume". If not, hit the command line and do a:
    
    ```
    mount
    ```
    
    which should show where the drive is mounted:
    
    ```
    /dev/sdb1 on /media/disk type vfat (rw,nosuid,nodev,shortname=mixed,uid=1000,utf8,umask=077,usefree)
    ```
    
6. Now that we know where the drive is mounted, let's unmount it:
    
    ```
    sudo umount /media/disk
    ```
    
7. Remove the CF card from your computer and insert it into the CF adapter on the ALIX board.
8. Connect one end of the null-modem cable to your computer's serial port and the other end to the serial port on the ALIX.
9. Fire up your favorite terminal emulation software such as minicom (or Hyperterminal on Windows) and use the following settings:
    - Baud rate: 38,400
    - Data: 8 bit
    - Parity: None
    - Stop: 1 bit
    - Flow control: None
    - Terminal: ANSI
10. Now apply power to the ALIX. If you are connected correctly, you should start to see the ALIX BIOS text.
11. While the BIOS is going through the memory test press the "s" key to enter the BIOS setup.
12. If have successfully entered the BIOS setup, you should see the text with some different options. Do the following:
     - Press "9" to set the baud rate at 9600
     - Press "q" to quit the BIOS setup
     - Press "y" to save the settings to flash
13. If you start seeing gibberish ASCI characters instead of text, then you need to set your terminal emulation software to 9600 baud instead of the 38,400 we set it at earlier.
14. Now reboot the ALIX by power cycling the unit (unplug the power, plug it back in).
15. With the terminal set to 9600 baud, we should see the boot-up process and if all is well it should look akin to a Free-BSD boot.
16. If all goes well and pfSense discovers your hardware, then you are good to go. To get started, you need to:
     - Assign the interfaces
     - Give the LAN interface an IP address that works for your internal network (i.e. 192.168.1.1)
17. Once you plugged the LAN interface into your network, then fire up your web browser and surf on over to the IP address you gave for the LAN interface (http://192.168.1.1)

## Updating the ALIX BIOS

Unfortunately for me pfSense did not properly detect the NIC modules. After some digging around on the 'net it looked like a BIOS upgrade would do the trick as the version on my ALIX board was 0.98b. The latest and greatest was 0.99. So here is how to flash upgrade the BIOS:

1. Download the [FreeDOS bootable image](http://www.pcengines.ch/file/freedos3.zip) from PC Engine's website.
2. Insert another CF card into your computer. If you only have the one CF card, then you'll have to re-do the previous instructions to get the pfSense image back on the card after you're done updating the BIOS.
3. Unzip the image from the download if necessary (it was a Zip file when I downloaded it)
    
    ```
    unzip freedos3.zip Archive:  freedos3.zip
      inflating: freedos_alixupdate_0.99.img
    ```
    
4. Now write this image to the CF card:
    
    ```
    dd if=freedos_alixupdate_0.99.img of=/dev/sdb
    ```
    
5. Unmount your CF card from the computer, and insert it into the ALIX.
6. Power on the ALIX and press "s" to enter the BIOS setup.
7. Change the drive configuration to LBA by pressing "L".
8. Now press "q" to exit, and "y" to save your changes to flash.
9. The system should boot the FreeDOS image and automatically run the BIOS flash utility (sb.com) to reprogram the flash.
10. Once it is done, power off the ALIX and reinsert your pfSense imaged CF card. If you only had one card you'll now have to go back and rewrite the pfSense image to the card.
