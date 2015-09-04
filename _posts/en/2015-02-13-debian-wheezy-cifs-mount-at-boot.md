---
layout: post
title:  "debian-wheezy-cifs-mount-at-boot"
date: 2015-02-13 00:00:00
name: debian-wheezy-cifs-mount-at-boot
---

I passed some hours today (rebooting ten times a server can be frustrating ...) to search why the cifs share were not mount at boot on a one of my servers.

The mount command was successful when I launched it manually.

[A comment of this post](http://www.turnkeylinux.org/forum/support/20130814/nfs-share-not-mounting-boot) recommend to disable the asynchrone mode but it didn't work for me.

I suspect the problem come from the fact we use a bridge interface for the network, and the mount operations are maybe launched before the "real" ready state of the interface

So I add a sleep time to the ``/etc/network/if-up.d/mountnfs`` script.


    mount -a -t$NETFS

become

    sleep 30
    mount -a -t$NETFS

and it works ...
