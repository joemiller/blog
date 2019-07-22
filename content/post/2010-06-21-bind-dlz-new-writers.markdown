---
author: "Joe Miller"

draft: true




categories:
  - software
comments: true
date: 2010-06-21 19:56:55 -0700
date_gmt: 2010-06-22 02:56:55 -0700
published: true
status: publish
tags:
  - dns
  - perl
title: New writer modules for BIND-DLZ's DNS Performance Testing Tools
url: /2010/06/21/bind-dlz-new-writers/


---

The [BIND-DLZ](http://bind-dlz.sourceforge.net/ "BIND-DLZ") project publishes an excellent set of performance testing [tools](http://bind-dlz.sourceforge.net/perf_tools.html "bind-dlz performance testing tools")which make it easy to generate a lot of fake DNS data for a variety of DNS server types.

I have extended this excellent tool set by creating a few new "writer" modules:

- contrib::writers::mydns::file
- contrib::writers::powerdns::xdb
- contrib::writers::powerdns::sqlite3
- contrib::writers::powerdns::gmysql
- contrib::writers::windowsdns::file

<!--more-->

Additionally, I modified the original **dnsDataGen.pl** script slightly so that the SOA record is always created first when generating a new zone.  This was necessary for most of the new writer modules to function properly.

- [DLZPerfTools-1.1-jmiller-contrib.tar.gz](http://www.joeym.net/files/bind-dlz-patches/DLZPerfTools-1.1-jmiller-contrib.tar.gz "DLZPerfTools-1.1-jmiller-contrib.tar.gz") -The original upstream version of DLZPerfTools-1.1 with my new modules and changes already applied.
- [bind-dlz-jmiller.patch](http://www.joeym.net/files/bind-dlz-patches/bind-dlz-jmiller.patch "bind-dlz-jmiller.patch") - A diff file that can be applied against the upstream DLZPerfTools-1.1.tar.gz
- [sample.conf](http://www.joeym.net/files/bind-dlz-patches/sample.conf "sample.conf") - A sample config file for use with dnsDataGen.pl demonstrating how to use each of the new writer modules.
- [dns-dict](http://www.joeym.net/files/bind-dlz-patches/dns-dict "dns-dict") - My personal list of random words that I use to feed into dnsDataGen.pl
- [announcement.txt](http://www.joeym.net/files/bind-dlz-patches/announcement.txt "announcement.txt") -Announcement email to the bind-dlz-testers mailing list. This has a little bit more info about the patch and the new writer modules, if you need it.