---
layout: post
title: Shorewall, ulogd and elk
name: shorewall-ulogd-elk
---

This article will explain what you can do with [shorewall](http://www.shorewall.net), a gateway/firewall configuration tool for GNU/Linux and the [elk stack](https://www.elastic.co/products).

![Shorewall logo](/pub/shorewall/shorewall-logo.png)

![shorewall-kibana](/pub/shorewall/shorewall-kibana.png)


Shorewall is a gateway/firewall configuration tool for GNU/Linux. It has a very good documentation and is very easy to configure.

## Connection Tracking

Connection Tracking allow you to keep track of your network sessions. You can have different states for a network packet: NEW, ESTABLISHED, RELATED, INVALID, UNTRACKED.

## ulogd
[Ulogd](http://www.netfilter.org/projects/ulogd/) is a userspace logging daemon for netfilter/iptables related logging. It let you log informations in different file output format like CSV or JSON or even in databases.

JSON log files are useful to work with elk [filebeat](https://www.elastic.co/products/beats/filebeat).

Debian packages:
    - ulogd2
    - ulogd2-json

/etc/ulogd.conf
{% highlight properties %}
...
[global]
...
#Use json plugin
plugin="/usr/lib/x86_64-linux-gnu/ulogd/ulogd_output_JSON.so"
...
# Get netfilter conntrack data and use emunfct1 with syslog like format
stack=ct1:NFCT,ip2str1:IP2STR,print1:PRINTFLOW,emunfct1:LOGEMU

# Get netfilter conntrack data and use jsonnfct1 with JSON format
stack=ct2:NFCT,ip2str1:IP2STR,jsonnfct1:JSON
...
[ct1]
event_mask=0x00000001
hash_enable=0
...
[ct2]
event_mask=0x00000004
hash_enable=0
...
[emunfct1]
file="/data/ulog/ulogd_nfct.log"
sync=1

[jsonnfct1]
sync=1
file="/data/ulog/ulogd_nfct.json"
...
{% endhighlight %}

Then you configure filebeat to read the json file:
/etc/filebeat# view filebeat.yml 
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

Logstash filter config for filebeat json.
conf.d/10-ulog-filter.conf
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

Note we have to add a ruby code to replace all dot characters by another one because elasticsearch don't like them on the key indices.

Thanks to  Eric Leblond  for these posts: [https://home.regit.org/2014/02/using-ulogd-and-json-output/](https://home.regit.org/2014/02/using-ulogd-and-json-output/)
[https://home.regit.org/2014/02/logging-connection-tracking-event-with-ulogd/](https://home.regit.org/2014/02/logging-connection-tracking-event-with-ulogd/)
