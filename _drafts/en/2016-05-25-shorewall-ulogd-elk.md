---
layout: post
title: How to send Shorewall logs to elk
name: shorewall-ulogd-elk
---

This article will explain how to send NetFilter/[shorewall](http://www.shorewall.net) tracked connections logs made with [Ulogd](http://www.netfilter.org/projects/ulogd/) to the [elk stack](https://www.elastic.co/products). You can then use kibana to search for specific activity from your network.

<!-- ![Shorewall logo](/pub/shorewall/shorewall-logo.png) -->

What we use:

- [Shorewall](http://www.shorewall.net) is a gateway/firewall configuration tool for GNU/Linux. It has a very good documentation and is very easy to configure.
- [Ulogd](http://www.netfilter.org/projects/ulogd/) is a userspace logging daemon for netfilter/iptables related logging. It lets you log information in different file output format like CSV or JSON or even in databases.
- [The ELK stack](https://www.elastic.co): [Elasticsearch](https://www.elastic.co/products/elasticsearch) + [Logstash](https://www.elastic.co/products/logstash) + [Kibana](https://www.elastic.co/products/kibana) + [Filebeat](https://www.elastic.co/products/beats/filebeat).

Logging every packets going through Shorewall is not a very good idea since, depending on your use case, this could result on very large log files size. We can reduce the information using [Connection Tracking](https://en.wikipedia.org/wiki/Netfilter#Connection_tracking) and log only network sessions.

We will set up a JSON log file with Ulogd and read it with Filebeat.

## Ulogd

You can find a good blog post on Ulogd and Json [here](https://home.regit.org/2014/02/using-ulogd-and-json-output/).

Here is my ulogd.conf file made from [this blog post](https://home.regit.org/2014/02/using-ulogd-and-json-output/):

```/etc/ulogd.conf```
{% highlight properties %}
...
[global]
...
#Use json plugin
plugin="/usr/lib/x86_64-linux-gnu/ulogd/ulogd_output_JSON.so"
...
# Get netfilter conntrack data and use jsonnfct1 with JSON format
stack=ct2:NFCT,ip2str1:IP2STR,jsonnfct1:JSON
...
[ct2]
event_mask=0x00000004
hash_enable=0
...
[jsonnfct1]
sync=1
file="/var/log/ulogd_nfct.json"
...
{% endhighlight %}

## Filebeat

We configure filebeat to read the json file:

```/etc/filebeat```
{% highlight yaml %}
filebeat:
  prospectors:
    -
      paths:
        - /data/ulog/ulogd_nfct.json
      input_type: log
      document_type: ulog
...
{% endhighlight %}

On the other side we need a logstash filter configuration to specify the json configuration and fix a bug: we have to add a ruby code to replace all dot characters by another one because elasticsearch don't like them on the key indices.

```conf.d/10-ulog-filter.conf```
{% highlight text %}
filter {
  if [type] == "ulog" {
    json{
      source => "message"
    }
    ruby {
      code => "event.to_hash.keys.each { |k| event[ k.gsub('.','_') ] = event.remove(k) if k.include?'.' }"
    }
  }
}
{% endhighlight %}

## Kibana

Here is the result:

![shorewall-kibana](/pub/shorewall/shorewall-kibana.png)

Thanks to  Eric Leblond  for his posts [using-ulogd-and-json-output](https://home.regit.org/2014/02/using-ulogd-and-json-output/) and [logging-connection-tracking-event-with-ulogd](https://home.regit.org/2014/02/logging-connection-tracking-event-with-ulogd/).
