---
author: "Joe Miller"
tags:
  - software
  - monitoring
  - graphite
comments: true
date: 2011-09-21 11:34:46 -0700
published: true
title: List of statsd server implementations
---

Statsd is a simple client/server mechanism from the folks at Etsy that allows operations and development teams to easily feed a variety of metrics into a Graphite system. For more info on statsd read the seminal blog article on Statsd ["Measure Anything, Measure Everything"](http://codeascraft.etsy.com/2011/02/15/measure-anything-measure-everything/ "Etsy: Measure Anything, Measure Everything").

<!--more-->

As would be expected there are statsd clients in many languages. But, there are also many implementations of the statsd server. This is nice because each organization can pick the one that best fits them. For example, a python shop might prefer to deploy a python based statsd instead of Etsy's original node.js implementation. Also, there are some statsd implementations that diverge from the original design and provide additional features.

I could not find a single resource that listed all of the different implementations, so I figured I would try to start one here.

- [Flickr's StatsD](https://github.com/iamcal/Flickr-StatsD "https://github.com/iamcal/Flickr-StatsD"): Perl. This is the real original statsd which inspired the Etsy StatsD. It's here for historical purposes and is not recommended for production use.
- [Etsy's statsd](https://github.com/etsy/statsd "https://github.com/etsy/statsd"): node.js. The (new) Original.
- [petef-statsd](https://github.com/fetep/ruby-statsd "https://github.com/fetep/ruby-statsd"): ruby. Supports AMQP.
- [statsd\_rb](https://github.com/seatgeek/statsd_rb "https://github.com/seatgeek/statsd\_rb"): ruby.
- [quasor/statsd](https://github.com/quasor/statsd "https://github.com/quasor/statsd"): ruby. can send data to graphite or mongoDB
- [py-statsd](https://github.com/sivy/py-statsd "py-statsd"): python (including python client code).
- [zbx-statsd](https://github.com/pistolero/zbx-statsd "https://github.com/pistolero/zbx-statsd"): python, based on py-statsd. Sends data to Zabbix instead of graphite.
- [statsd.scala](https://github.com/jamesgolick/statsd.scala "https://github.com/jamesgolick/statsd.scala"): scala. Sends data to Ganglia instead of Graphite. Different messaging protocol, uses JSON.
- [txStatsD](https://launchpad.net/txstatsd "https://launchpad.net/txstatsd"): python + twisted, from the folks @ Canonical
- [statsd-librato](https://github.com/engineyard/statsd-librato "https://github.com/engineyard/statsd-librato"): node.js. Fork of etsy's statsd for sending data to Librato instead of graphite from the folks @ Engine Yard.
- [estatsd](https://github.com/opscode/estatsd "https://github.com/opscode/estatsd"): erlang. From the folks @ Opscode
- [metricsd](https://github.com/mojodna/metricsd "https://github.com/mojodna/metricsd"): scala. Should be drop-in compatible with etsy's statsd, but with support for additional metric types (eg: meter, gauge, histogram)
- [statsd-c](https://github.com/jbuchbinder/statsd-c "statsd-c"): C. compatible with original etsy statsd
- [statsd (librato)](https://github.com/librato/statsd "statsd (librato)"): node.js.   [Librato's](http://librato.com/ "Librato") officially maintained fork of statsd based on the changes from Engine Yard. Supports multiple graphing services including [Librato Metrics](http://metrics.librato.com "Librato Metrics")
- [bucky](https://github.com/cloudant/bucky "bucky"): python. A unique spin on statsd that supports collecting data from statsd clients, collectd, and metricsd, with output to graphite. The ability to translate collectd plugin names to be more graphite-friendly is very compelling.
- [clj-statsd-svr](https://github.com/netmelody/clj-statsd-svr "clj-statsd-svr"): Clojure.
- [statsite](https://github.com/armon/statsite "statsite"): C. Statsite is designed to be both highly performant, and very flexible, using libev to be extremely fast.
- [statsdaemon](https://github.com/bitly/statsdaemon "statsdaemon"): Go. Statsdaemon written in Go, from bitly.

Other interesting statsd-like projects that are not protocol compatible with the original Etsy statsd but may offer compelling features not found in other implementations:

- [estatsd (opscode)](https://github.com/opscode/estatsd): Erlang. Inspired by etsy's statsd but not protocol compatible.
- [estatsd (fauxsoup)](https://github.com/fauxsoup/estatsd): Erlang. Fork of opscode's estatsd with a focus on multi-datacenters and high-scalability.

Deprecated:

- [statsite](https://github.com/kiip/statsite/ "https://github.com/kiip/statsite/"): python. Replaced by a new implementation in C, see above.

Please leave a comment if you have an implementation that should be listed here. Feedback on any of the above implementations would be helpful too.
