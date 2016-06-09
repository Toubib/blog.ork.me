---
layout: post
title:  "Add jboss metrics to grafana with snmp"
date:   2015-10-09 23:51:27
name: jboss-snmp-grafana
---

It's quite easy to send data to graphite and then make a dashboard on Grafana. Here is how to get data from jboss with snmp (for Jboss before version 7).

I assume you already have a graphite-statsd-grafana installation.

# Setup your snmp

First you have to enable the snmp adaptor service, see [The SNMP Adaptor JBoss Service](https://docs.jboss.org/author/display/MOBICENTS/The+SNMP+Adaptor+JBoss+Service) and [JBossSNMPAdapter](https://developer.jboss.org/wiki/JBossSNMPAdapter).
You can test it with snmpget (.1.2.3.4.1.1 is configured to get the active thread count)

~~~shell
# snmpget -v2c -t3 -r1 -c public localhost:20771 .1.2.3.4.1.1
iso.2.3.4.1.1 = INTEGER: 106
~~~

You should also enable snmp at the JVM level (see [docs.oracle.com](http://docs.oracle.com/javase/7/docs/technotes/guides/management/snmp.html) or [JVM_Monitoring_using_SNMP](http://www.opennms.org/wiki/JVM_Monitoring_using_SNMP))

~~~term
# snmpget -v2c -t3 -r1 -c public localhost:8071  iso.3.6.1.4.1.42.2.145.3.163.1.1.2.101.1.2.2 iso.3.6.1.4.1.42.2.145.3.163.1.1.2.101.1.3.2
iso.3.6.1.4.1.42.2.145.3.163.1.1.2.101.1.2.2 = Counter64: 143
iso.3.6.1.4.1.42.2.145.3.163.1.1.2.101.1.3.2 = Counter64: 9045
~~~

please be careful with the jboss snmp adaptor which is 32 bits only and can give you shit values sometimes like if you have more than 4GB of memory. Use JVM values instead !

# Send data to statsd

I wrote a daemon in order to query the snmp services and push the data to statsd.
