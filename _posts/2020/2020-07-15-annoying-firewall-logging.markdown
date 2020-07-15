---
layout: post
title:  "Annoying Firewall Logging"
date:   2020-07-15 11:08:20 +0200
---

Hej Hej!

I would like to share how to deal with ufw logging. Turn out ufw uses kernel logging facility. 
It makes the kernel log file will be flooded by ufw messages. Furthermore, if you type `dmesg`,
the output will also be flooded by ufw messages. 

There is someone who reported this issue as a bug [here](https://bugs.launchpad.net/ufw/+bug/1264621).
But I have no idea if this has been resolved or not. But yeah, I agree with someone there who say why ufw uses two, even three logging mechanisms. 
One logs the events in `/var/log/ufw.log`, one in `/var/log/kern.log`, and one when you type `dmesg`.

Well to deal with this annoying condition, at least this is annoying for me :p, let me share these two steps:
1. Modify `.bashrc` so it will omit “ufw” strings.
Just add `alias dmesg="dmesg | sed '/UFW/d'"` into the file. And try to type `dmesg` on the console after re-source the modified `.bashrc`.
2. Modify `/etc/rsyslog.d/20-ufw.conf` and just uncomment the last line. There is a tiny guide in the config file.

Okay then. Hopefully those steps works for you and you get a better feeling, I mean you will not be annoyed by ufw logging anymore 🙂

Cheers,\
Rahmanu

