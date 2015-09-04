---
layout: post
title:  "Fix pty /dev/ptmx bad permissions on Debian/lxc"
date:   2015-02-12 00:00:00
name:   debian-lxc-tty
---

I use rlfe with sqlplus in order to have a better sql cli:
lfe -h /home/`whoami`/.sqlhistory $SQLPLUS $conn | tee -a $OUTFILE

But I had this problem within an lxc container:

    ~$ rlfe 
    ptypair: could not open master pty: No such file or directory

This init script resolve the issue:

/etc/init.d/fix-ptmx-perm

    #! /bin/sh
    
    ### BEGIN INIT INFO
    # Provides:          ptmx device workaround
    # Required-Start:    
    # Required-Stop:
    # X-Start-Before:    
    # Default-Start:     2 3 4 5
    # Default-Stop:      
    # Short-Description: ptmx device workaround
    # Description: ptmx device workaround
    ### END INIT INFO
    
    chmod 666 /dev/ptmx

To add it on boot:

    update-rc.d fix-ptmx-perm defaults
