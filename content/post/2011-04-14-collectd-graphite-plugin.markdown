---
author: "Joe Miller"





categories:
  - software
comments: true
date: 2011-04-14 11:27:19 -0700
date_gmt: 2011-04-14 18:27:19 -0700
published: true
status: publish
tags: []
title: Collectd-Graphite plugin.  Bringing together two great tools
url: /2011/04/14/collectd-graphite-plugin/


---

[Collectd](http://collectd.org/ "collectd.org") is a powerful tool for gathering metrics using its wide range of plugins, such as cpu, disk, load, memory, etc.  But there is a lack of good frontend tools for visualizing the data collectd produces.

[Graphite](http://graphite.wikidot.com/ "graphite") is an amazingly powerful tool from Orbitz for visualizing metrics, but there is a lack of tools for gathering host-level stats and sending into graphite.

It would be great if we could leverage the strengths of both tools.

<!--more-->

So, I wrote a plugin for collectd that will send data into a graphite instance - **[collectd-graphite](https://github.com/joemiller/collectd-graphite "collectd-graphite")** . This plugin runs inside the collectd process using the collectd-perl interface. This makes it different than Jordan Sissel's [collectd-to-graphite](https://github.com/loggly/collectd-to-graphite) tool which runs in a separate process using node.js. The plugin-based approach reduces the number of moving parts we need to worry about (ie: less stuff to monitor, fail, restart, etc.) Jordan's tool is good, btw, but I wanted something different.

Grab the plugin on github:   [https://github.com/joemiller/collectd-graphite](https://github.com/joemiller/collectd-graphite "collectd-graphite")

In my setup, I have collectd running on all hosts with the cpu, disk, memory, and network-client plugins loaded.  The network-client is configured to send data to a central collectd server which also runs my graphite processes.  The collectd-graphite plugin is loaded on this machine only and relays all data received by collectd to graphite.

Alternatively, you could load the collectd-graphite plugin on every machine in your environment and send data directly to graphite.  This may not scale well, however, and you would not get any of the security that can be applied with collectd's network protocol.

**Update Sept 24, 2012** : Collectd 5.1+ has an integrated C-based graphite plugin now. I would recommend using that as it will be supported and maintained as part of Collectd core going forward. Info here: [http://collectd.org/wiki/index.php/Plugin:Write\_Graphite](http://collectd.org/wiki/index.php/Plugin:Write_Graphite "http://collectd.org/wiki/index.php/Plugin:Write\_Graphite")
