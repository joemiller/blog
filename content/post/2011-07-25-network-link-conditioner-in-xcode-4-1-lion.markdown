---
author: "Joe Miller"
tags:
  - macosx
  - testing
comments: true
date: 2011-07-25 15:07:42 -0700
date_gmt: 2011-07-25 22:07:42 -0700
draft: true
title: Network Link Conditioner in Xcode 4.1, Lion
---

Previously, I wrote a [post](http://joemiller.me/2010/08/31/simulate-network-latency-packet-loss-and-bandwidth-on-mac-osx/ "Simulate network latency, packet loss, and low bandwidth on Mac OSX") about using the 'dummynet' functionality in Mac OSX's ipfw(8) firewall to simulate a variety of networking conditions, such as:  bandwidth, packet loss, latency (delay). This is a great feature for testing software under a variety of network conditions but it can be a little tough to use unless you're comfortable at the command line, or even better, have unix scripting skills since there are multiple commands required to create even simple scenarios.

Then, today I noticed that Apple now includes a new prefPane in Xcode 4.1 and Lion called "Network Link Conditioner" that simplifies all of this, and even includes a few profiles to get you started (eg: "Wifi, Average case", "3G, Lossy Network".) Pretty cool feature. Especially useful for iOS developers. Screenshot below.

<!--more-->

- Install: find and run /Developer/Applications/Utilities/Network Link Conditioner/Network Link Conditioner.prefPane

[![](/images/Screen-Shot-2011-07-25-at-2.57.41-PM.png "Network Link Conditioner")](/images/Screen-Shot-2011-07-25-at-2.57.41-PM.png)

 
