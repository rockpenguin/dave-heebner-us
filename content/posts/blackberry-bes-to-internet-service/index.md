---
title: "Blackberry BES to Internet Service"
date: 2008-02-02
categories: 
  - "recipes"
tags: 
  - "blackberry"
  - "exchange"
  - "outlook"
---

Cancelled my hosted Exchange account. It was cool, but I was only testing. Anyhoo, I had to purge Outlook and the BB of the BES settings. So, here's what seems to have worked:

1. Before I cancelled the hosted Exchange account, I exported the data from Outlook - just to be safe!
2. Control Panel -> Mail -> blew away the profile. Probably not necessary, but I had a backup of the data in case...
3. Created a new Outlook profile with regular ol' POP3 mail (oh, when will Microsoft ever get IMAP4 right. Strikeout deleted mail??? WTF!)
4. Uninstalled BB Desktop Manager software. Again, probably not necessary but I figured what the hell.
5. Reinstalled BB Desktop Manager and chose only Blackberry Internet Service this go round.
6. Went into BB Desktop Manager and configured the requisite settings to synch with Outlook.
7. Went into the following apps -> Options on the BB itself and set the "Wireless Sync" to **NO**.Address Book, Tasks, Memo

That seems to have done the trick. Still receiving messages on my Internet mail accounts, so that's good.

Update: 2008-02-03

Kept getting these messages on the BB, and oddly enough, in one of my IMAP accounts:

```
This message is used to carry data between the BlackBerry handheld and an associated server. Please do not delete, move or respond to this message - it will be processed by the server.

BEGINETP 111
ENDETP -1946816486
```

Turns out BES was still in the mix somewhere. I tried resending the Service Books to no avail. The trick that worked was to delete the Service Book called "Desktop \[OTA...\]". I can't remember exactly what was inside the brackets, but it had something to do with OTA (over the air) synchronization. Once I deleted that service book, I went into the BB email setup and resent the service books. Viola!
