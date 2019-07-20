---
author: "Joe Miller"





categories:
  - testing
  - dns
comments: true
date: 2010-12-22 19:53:20 -0800
date_gmt: 2010-12-23 02:53:20 -0800
published: true
status: publish
tags: []
title: dns_compare.py - testing DNS servers
url: /2010/12/22/dns_compare-py-testing-dns-servers/


---

Sometimes it might be nice to be able to test a DNS server's output, such as with a continuous monitoring system, or as a validation tool when migrating to a different DNS server (or service.)

{{< highlight text >}}
$ dns_compare.py -z example.com --file example.com.zone --server 10.1.1.1
....X...X..done
Results:
Matches: 9
Mis-matches: 2
{{< / highlight >}}

To see more examples or download the code, goto: [https://github.com/joemiller/dns\_compare](https://github.com/joemiller/dns_compare "https://github.com/joemiller/dns\_compare")
