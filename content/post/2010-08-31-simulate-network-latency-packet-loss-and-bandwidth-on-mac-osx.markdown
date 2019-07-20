---
author: "Joe Miller"





categories:
  - software
  - macosx
  - testing
comments: true
date: 2010-08-31 16:40:48 -0700
date_gmt: 2010-08-31 23:40:48 -0700
published: true
status: publish
tags: []
title: Simulate network latency, packet loss, and low bandwidth on Mac OSX
url: /2010/08/31/simulate-network-latency-packet-loss-and-bandwidth-on-mac-osx/


---

Sometimes while testing you may want to be able to simulate network latency, or packet loss, or low bandwidth. I have done this with Linux and  [tc/netem](http://www.linuxfoundation.org/collaborate/workgroups/networking/netem "tc/netem") as well as with Shunra on Windows, but I had never done it on Mac OSX.

It turns out that Mac OSX includes 'dummynet' from FreeBSD which has the capability to do this WAN simulation.

<!--more-->

Here is a quick example:

- Inject 250ms latency and 10% packet loss on connections between my workstation and my development web server (10.0.0.1)
- Simulate maximum bandwidth of 1Mbps

{{< highlight text >}}
# Create 2 pipes and assigned traffic to and from our webserver to each:
$ sudo  ipfw add pipe 1 ip from any to 10.0.0.1
$ sudo  ipfw add pipe 2 ip from 10.0.0.1 to any


# Configure the pipes we just created:
$ sudo ipfw pipe 1 config delay 250ms bw 1Mbit/s plr 0.1
$ sudo ipfw pipe 2 config delay 250ms bw 1Mbit/s plr 0.1
{{< / highlight >}}

A quick test:

{{< highlight text >}}
$ ping 10.0.0.1
PING 10.0.0.1 (10.0.0.1): 56 data bytes
64 bytes from 10.0.0.1: icmp_seq=0 ttl=63 time=515.939 ms
64 bytes from 10.0.0.1: icmp_seq=1 ttl=63 time=519.864 ms
64 bytes from 10.0.0.1: icmp_seq=2 ttl=63 time=521.785 ms
Request timeout for icmp_seq 3
64 bytes from 10.0.0.1: icmp_seq=4 ttl=63 time=524.461 ms
{{< / highlight >}}

Disable:

{{< highlight text >}}
$sudo ipfw list |grep pipe
  01900 pipe 1 ip from any to 10.13.1.133 out
  02000 pipe 2 ip from 10.13.1.133 to any in
$ sudo ipfw delete 01900
$ sudo ipfw delete 02000


# or, flush all ipfw rules, not just our pipes
$ sudo ipfw -q flush
{{< / highlight >}}

Notice that the round-trip on the ping is ~500ms.  That is because we applied a 250ms latency to both pipes, incoming and outgoing traffic.

Our example was very simple, but you can get quite complex since "pipes" are applied to traffic using standard ipfw firewall rules.  For example, you could specify different latency based on port, host, network, etc.

Packet loss is configured with the "plr" command.  Valid values are 0 - 1.  In our example above we used 0.1 which equals 10% packetloss.

This is a very handy way for developers on Mac's to test their applications in a variety of network environments.  And you get it for FREE.  On Windows you need to buy a commercial tool to achieve this (at least that was true the last time I looked, in 2008.)
