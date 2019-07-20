---
author: "Joe Miller"





categories:
  - internet services
comments: true
date: 2010-06-29 17:24:30 -0700
date_gmt: 2010-06-30 00:24:30 -0700
published: true
status: publish
tags:
  - dns
title: Free IPv4 + IPv6 DNS hosting
url: /2010/06/29/free-ipv4-ipv6-dns-hosting/


---

I have been a big fan of [TunnelBroker.net](http://tunnelbroker.net/ "tunnelbroker.net")'s IPv6 tunneling service for a number of years now, but I was pleasantly surprised to discover a new service from TunnelBroker.net (HE - Hurrican Electric) -- [IPv6 DNS hosting](https://dns.he.net/ "free ipv6 dns hosting")!  Actually it's dual-stack DNS hosting, allowing you to serve DNS requests for your domains to both the IPv4 and IPv6 internet.

It is currently in "open beta" status and available to anyone with a (free) ipv6 tunnelbroker.net account, or if you happen to be a customer of the parent company providing this service, [HE](http://he.net "hurricane electric") (Hurricane Electric.)

<!--more-->

The interface isn't the most user-friendly in the world, but if you're a little bit technical and have a basic understanding of DNS you will be fine.  If you've ever used the web interfaces of a registrar like GoDaddy to manage your DNS settings this will be familiar.

Here are some of the cool aspects of this service:

- Updates are visibile very quickly, within seconds I would say
- Supports reverse DNS for both ipv4 and ipv6 addresses
- Multiple domains per account (25 actually)
- Supported record types:  A, AAAA, PTR, TXT, SRV, CNAME, MX, NS
- Multiple TTL's allowed:  5 min, 15 min, 1 hour, 2 hour, 4 hour, 8 hour, 12 hour, 24 hour (1 minute is grayed out, maybe it's coming soon)
- No DNSSEC support yet, but listed as "coming (3-6 months)"
- Slave zones supported
- Export in BIND format for easy backup
- No anycast, but it looks like their DNS servers are spread across 5 geographies (ns1 - ns5.he.net)

It would be nice if it offered an import function which would help with migrating into the service, but luckily for me I only had a few records to move over.

Very nice.
