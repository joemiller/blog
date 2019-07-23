---
author: "Joe Miller"
draft: true





categories:
  - devOps
  - monitoring
  - puppet
comments: true
date: 2011-11-05 19:09:15 -0700
date_gmt: 2011-11-06 02:09:15 -0800
status: publish
tags:
  - metrics
title: Correlating Puppet changes to events in your infrastructure using graphite
url: /2011/11/05/correlating-puppet-changes-to-events-in-your-infrastructure/


---

Sometimes it is pretty obvious when Puppet changes something in your infrastructure and bad things happen in a big dramatic way. Other times it's not so obvious. It can be invaluable to be able to correlate changes made by Puppet to other events happening in your infrastructure.

<!--more-->
For example, in this diagram we have plotted the load average from a group of servers. Blue vertical lines mark points in time when puppet modified a resource on a host in the group. We can see that immediately following a puppet change the load spiked on one of the servers.

[![](/images/graphite-puppet-event-example-720x300-crop.png "graphite-puppet-event-example-720x300-crop")](/images/graphite-puppet-event-example-720x300-crop.png)

Code available on github:

- [https://github.com/joemiller/puppet-graphite\_event](https://github.com/joemiller/puppet-graphite_event "https://github.com/joemiller/puppet-graphite\_event")
