---
author: "Joe Miller"

draft: true




categories:
  - software
  - devOps
  - howto
  - linux
  - monitoring
  - sensu
  - rabbitmq
  - graphite
comments: true
date: 2012-02-02 14:18:52 -0800
date_gmt: 2012-02-02 21:18:52 -0800
published: true
status: publish
tags: []
title: Sensu and Graphite
url: /2012/02/02/sensu-and-graphite/


---

> **Updated December 7, 2013** : I no longer recommend using the approach described in this post. Please read [Sensu and Graphite, Part 2](http://joemiller.me/2013/12/07/sensu-and-graphite-part-2/ "Sensu and Graphite, Part 2") instead.
> 
<!--more-->
> **Updated October 16, 2012** : Removed "passive":"true" from the graphite amqp handler definition in Sensu. This is too brittle. Sensu will fail to start unless graphite has started first and created the exchange on the RabbitMQ server. By matching the "durable":"true" setting that graphite expects, then we can start either service in any order.
> 
> **Updated October 16, 2012** : Updated graphite amqp handler definition to use the new "mutator"'s in Sensu 0.9.7. See the for details on backwards-incompatible changes as Sensu moves towards a 1.0.0 release.

It's been pretty exciting to see the number of folks getting involved with [Sensu](http://www.sonian.com/cloud-tools/cloud-monitoring-sensu/) lately, as judging by the increased activity on the #sensu channel on Freenode. One of the most common questions is how to integrate Sensu and Graphite. In this article I'll cover two approaches for pushing metrics from Sensu to Graphite.

Remember: think of Sensu as the "monitoring router". While we are going to show how to push metrics to Graphite, it is just as easy to push metrics to any other system - Librato, Cube, OpenTSDB, etc. In fact, it would not be difficult at all to push metrics to multiple graphing backends in a fanout manner.

### Install vmstat-metrics plugin

For this example, we're going to use the `vmstat-metrics` plugin which can be found in the [sensu-community-plugins](https://github.com/sonian/sensu-community-plugins) repository on github. The plugins in this repo are meant to be cherry-picked, so let's just grab the one we're interested in:

{{< highlight bash >}}
cd /etc/sensu/plugins/
sudo wget https://raw.github.com/sonian/sensu-community-plugins/master/plugins/system/vmstat-metrics.rb
sudo chmod +x vmstat-metrics.rb
{{< / highlight >}}

Most, but not all, of the plugins in the sensu-community-plugins repository use helper classes from the [sensu-plugin](https://github.com/sonian/sensu-plugin), so we'll need to install that gem on the nodes that will be running the `vmstat-metrics` plugin.

{{< highlight bash >}}
sudo gem install sensu-plugin --no-rdoc --no-ri
{{< / highlight >}}

Open up `vmstat-metrics.rb` and we see that it inherits a lot of plumbing from the `Sensu::Plugin::Metric::CLI::Graphite` class. Keep this in mind when writing your own metrics-gathering plugins, it can save you some time.

Let's run the plugin manually so we can see the output.

{{< highlight bash >}}
./vmstat-metrics.rb 
stats.swap.in 0 1328153991
stats.swap.out 0 1328153991
stats.memory.active 122160 1328153991
stats.memory.swap_used 8 1328153991
stats.memory.free 48556 1328153991
stats.memory.inactive 73704 1328153991
stats.cpu.waiting 1 1328153991
stats.cpu.idle 95 1328153991
stats.cpu.system 4 1328153991
...
{{< / highlight >}}

We will want to customize the path before sending this data to Graphite, including adding a hostname to differentiate it from other hosts. This can be done with the --scheme switch:

{{< highlight bash >}}
./vmstats-metrics.rb --scheme stats.`hostname -s`
stats.host01.swap.in 0 1328155423
stats.host01.swap.out 0 1328155423
stats.host01.memory.active 122512 1328155423
stats.host01.memory.swap_used 8 1328155423
stats.host01.memory.free 43856 1328155423
stats.host01.memory.inactive 75120 1328155423
stats.host01.cpu.waiting 1 1328155423
stats.host01.cpu.idle 95 1328155423
stats.host01.cpu.system 4 1328155423
...
{{< / highlight >}}

If you're familiar with Graphite, this should look familiar. This data fits Graphite's "stat.name value timestamp" format and we can be feed it directly into Graphite via a couple different methods.

### Create a check .json

Let's push out a check definition to our nodes running sensu-client and sensu-server nodes. Don't forget to install the plugin as well as the `sensu-plugin` gem.

File: `/etc/sensu/conf.d/metrics_vmstat.json`:

{{< highlight javascript >}}
{
  "checks": {
    "vmstat_metrics": {
      "type": "metric",
      "handlers": ["graphite"], 
      "command": "/etc/sensu/plugins/vmstat-metrics.rb --scheme stats.:::name:::",
      "interval": 60,
          "subscribers": ["webservers"]
    }
  }  
}
{{< / highlight >}}

There are two new items in this check definition that we haven't covered yet:

- 

The first is a new attribute: "type": "metric". This is critical. It tells the sensu-server to send the output of every invocation of this check to the specified handlers. Normally, only checks that return a non-zero exit status indicating a failed check will be passed onto handlers. For metrics, however, we always want to send the output to the handlers.

- 

The second is the use of a "custom variable" in the command: `:::name:::`. Sensu will replace this at execution time with the `name` attribute from the `client` section of the Sensu config. There's a lot of other stuff we can do with custom variables which will be covered in future blogs.

Next, we need to create a handler to do something with this data. Since we're using Graphite in this example, we'll explore two methods for getting this data into graphite: direct TCP and AMQP.

### Method 1 - Direct TCP (via netcat) handler

The first method will be to simply send this data to Graphite via TCP socket using netcat. This is a very simple approach and should work on most boxes (with netcat installed) and doesn't require configuring Graphite to use AMQP.

Fetch `graphite_tcp.rb` from the [sensu-community-plugins](https://github.com/sonian/sensu-community-plugins/tree/master/handlers/metrics) repo and copy to `/etc/sensu/handlers`.

Next create `/etc/sensu/conf.d/graphite_tcp.json` config file:

{{< highlight javascript >}}
{
  "graphite": {
    "server":"graphite.example.com",
    "port":"2003"
  }
}
{{< / highlight >}}

Create `/etc/sensu/conf.d/handler_graphite.json`:

{{< highlight javascript >}}
{
  "handlers": {
    "graphite": {
      "type": "pipe",
      "command": "/etc/sensu/handlers/graphite_tcp.rb"
    }
  }
}
{{< / highlight >}}

Restart sensu-server and you should start seeing vmstat metrics show up in Graphite.

There is a downside to this approach, however, and that is scalability. For each metric that is received, sensu-server will fork and execute this handler. With many nodes, many checks generating metrics, and a low interval, this could quickly add up to dozens or hundreds of short-lived processes being forked on the sensu-server.

### Method 2 - integrated AMQP handler

Remember that I keep calling Sensu "the monitoring router"? Because Sensu and Graphite both speak the AMQP messaging protocol, we can configure sensu-server to take the check output directly off of a rabbitmq queue and copy it to a new queue in a format for Graphite. This approach should be very fast and very scalable.

Configure Graphite's `carbon-cache` for AMQP:

`/opt/graphite/conf/carbon.conf`:

{{< highlight ini >}}
ENABLE_AMQP = True
# AMQP_VERBOSE = True
AMQP_HOST = sensu.example.com
AMQP_PORT = 5672
AMQP_VHOST = /sensu
AMQP_USER = sensu
AMQP_PASSWORD = mypass
AMQP_EXCHANGE = metrics
AMQP_METRIC_NAME_IN_BODY = True
{{< / highlight >}}

An important part of this config is `AMQP_METRIC_NAME_IN_BODY = True` which means Graphite will determine the metric name from the contents of the message rather than via the routing key. See [here](https://answers.launchpad.net/graphite/+question/163389) for more info.

Next, we configure a handler on the sensu-server that will send metrics to the queue Graphite is listening on:

Edit `/etc/sensu/conf.d/handler_graphite.json`:

{{< highlight javascript >}}
{
  "handlers": {
    "graphite": {
      "type": "amqp",
      "exchange": {
        "type": "topic",
        "name": "metrics",
        "durable": "true"
      },
      "mutator": "only_check_output"
    }
  }
}
{{< / highlight >}}

A few things to note here:

- "type": "amqp" : We're telling sensu-server that this handler will re-route the check output to an AMQP exchange.
- "durable": "true" : Graphite (carbon-cache) creates the exchange as a durable exchange. We need to match this setting otherwise we will get an error from Rabbit and the connection will fail.
- "mutator": "only\_check\_output" : This instructs sensu-server to only send the raw `output` (from stdout) returned by the check to the AMQP exchange. By default, sensu-server would send the entire JSON doc with other metadata, but Graphite wouldn't know what to do with that data.

### Debug tips:

For debugging the Graphite side of things, it can be helpful to enable `AMQP_VERBOSE = True` in `carbon.conf`. With this enabled, the following will be visible in `listener.log` when a metric is successfully retrieved from rabbitmq by Graphite:

{{< highlight bash >}}
02/02/2012 10:11:11 :: Message received: Method(name=deliver, id=60) ('graphite_consumer', 8, False, 'metrics', '') content = 
02/02/2012 10:11:11 :: Metric posted: test.sensu-server.swap.in 0 1328195471
02/02/2012 10:11:11 :: Metric posted: test.sensu-server.swap.out 0 1328195471
02/02/2012 10:11:11 :: Metric posted: test.sensu-server.memory.active 126856 1328195471
...
{{< / highlight >}}