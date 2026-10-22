---
title: "SSH - disable StrictHostChecking as well as writing key to known_hosts"
date: 2017-06-27
categories: 
  - "recipes"
  - "techsupport"
tags: 
  - "linux"
  - "mac"
  - "ssh"
  - "virtualization"
---

I often do a lot of work with temporary Vagrant, Docker, VMs, and I don't really want those host keys clogging up my ~/.ssh/known\_hosts file. So I just add the following alias to my shell "rc" file. In my case, I use zsh so it's ~/.zshrc :

```
alias ssh-nocheck="ssh -o 'StrictHostKeyChecking no' -o 'UserKnownHostsFile /dev/null'"
```

Viola!  Now all I need to is to use something akin to:

```
ssh-nocheck -p 2222 root@192.168.2.138
```
