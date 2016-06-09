---
layout: post
title: How to graph shorewall accounting data
name: shorewall-graphite-grafana
---

This article will explain what you can do with [shorewall](http://www.shorewall.net), a gateway/firewall configuration tool for GNU/Linux, and [graphite](http://graphite.readthedocs.io/en/latest/overview.html)/[grafana](http://grafana.org/).

![Shorewall logo](/pub/shorewall/shorewall-logo.png)

Shorewall is a gateway/firewall configuration tool for GNU/Linux. It has a very good documentation and is very easy to configure.

In shorewall configuration you can create packet and byte counters with [accounting rules](http://shorewall.net/Accounting.html) so you can gather data precisely on what you need. An interesting option is to use the [xtables netfilter addon](http://shorewall.net/Accounting.html#perIP) who will create per IP metrics.

My /etc/shorewall/accounting config:
{% highlight text %}
#ACTION					CHAIN	SOURCE	DEST	PROTO   DPORT   SPORT   USER    MARK    IPSEC
ACCOUNT(loc-net,x.x.0.0/20)		-	eth2    eth1
{% endhighlight %}

We can then print counters with the "shorewall show ipa" command:

{% highlight text %}
host:~# shorewall show ipa
Shorewall 4.6.4.3 per-IP Accounting at host - 

Showing table: loc-net
IP: x.x.0.4 SRC packets: 480774 bytes: 49865249 DST packets: 0 bytes: 0
IP: x.x.1.100 SRC packets: 33941 bytes: 1899866 DST packets: 0 bytes: 0
IP: x.x.2.47 SRC packets: 5736 bytes: 1204376 DST packets: 0 bytes: 0
IP: x.x.2.51 SRC packets: 608394 bytes: 139710104 DST packets: 0 bytes: 0
IP: x.x.2.60 SRC packets: 141483 bytes: 30495738 DST packets: 0 bytes: 0
IP: x.x.2.69 SRC packets: 26145 bytes: 3335664 DST packets: 0 bytes: 0
IP: x.x.2.78 SRC packets: 285086 bytes: 68623864 DST packets: 0 bytes: 0
IP: x.x.2.81 SRC packets: 163715 bytes: 11279400 DST packets: 0 bytes: 0
{% endhighlight %}

Great ! All we have to do now is to send this data to graphite and we will write a small python script for that.

We will use the [daemonize](https://pypi.python.org/pypi/daemonize/) library to run our process in background (you can install it with "pip install daemonize").

{% gist Toubib/e1004ace58c3b4a3446b1e4e4cc59ada %}

You can launch it with the systemd shorewall-ipa-graphite.service unit file.

Finally we will create a grafana dashboard with some collectd metrics (ping and network interfaces) and our shorewall-ipa-graphite.py script metrics in a top ten consumers graphic:

![shorewall-grafana](/pub/shorewall/shorewall-grafana-2-f.png)

Here is the Metric configuration line, with a Transform: negative-Y series specific overrides on /tx.*/ to separate in and out:

{% highlight text %}
aliasSub(absolute(derivative(consolidateBy(sortByMaxima(highestCurrent(firewall.hostname.accounting.perip.*.*_bytes, 10)), 'max'))), '.*perip\.([0-9]*)_([0-9]*)_([0-9]*)_([0-9]*)\.([a-z]*).*', '\5.\3.\4')
{% endhighlight %}

Thanks to  Eric Leblond  for these posts: [https://home.regit.org/2014/02/using-ulogd-and-json-output/](https://home.regit.org/2014/02/using-ulogd-and-json-output/)
[https://home.regit.org/2014/02/logging-connection-tracking-event-with-ulogd/](https://home.regit.org/2014/02/logging-connection-tracking-event-with-ulogd/)
