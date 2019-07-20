---
author: "Joe Miller"





categories:
  - software
  - monitoring
  - groovy
comments: true
date: 2011-06-29 16:50:13 -0700
date_gmt: 2011-06-29 23:50:13 -0700
published: true
status: publish
tags: []
title: groovy-statsdclient
url: /2011/06/29/groovy-statsdclient/


---

In an attempt to learn some Groovy and Gradle I wrote an implementation of a Statsd client in groovy.  It's similar to other statsd clients in other languages and supports the typical increment(), decrement(), and timing() methods.

<!--more-->

It's not available on Maven Central or via Grape at this time, which will be a future learning exercise. In the meantime you can download a pre-built .jar or the source code from github:

- [https://github.com/joemiller/groovy-statsdclient](https://github.com/joemiller/groovy-statsdclient "https://github.com/joemiller/groovy-statsdclient")

For more background on Statsd, check out this blog article from Etsy:   [Measure Anything, Measure Everything](http://codeascraft.etsy.com/2011/02/15/measure-anything-measure-everything/ "Statsd: Measure anything, Measure Everything")

 
