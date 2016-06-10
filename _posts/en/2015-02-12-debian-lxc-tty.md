---
layout: post
title:  "Fix pty /dev/ptmx bad permissions on Debian/lxc"
date:   2015-02-12 00:00:00
name:   debian-lxc-tty
---

I use [`rlfe`](https://packages.debian.org/fr/wheezy/rlfe) with sqlplus in order to have a better sql cli:

~~~
rlfe -h /home/`whoami`/.sqlhistory $SQLPLUS $conn | tee -a $OUTFILE
~~~

But I had this problem within an lxc container:

~~~
~$ rlfe 
ptypair: could not open master pty: No such file or directory
~~~

The problem was a bad permission setting on /dev/ptmx. These permissions were set at boot so just change them on time wasn't enough.

This init script resolve the issue:

/etc/init.d/fix-ptmx-perm

{% highlight bash %}

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
{% endhighlight %}

To add it on boot:

~~~
~$ update-rc.d fix-ptmx-perm defaults
~~~
