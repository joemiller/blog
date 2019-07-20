---
author: "Joe Miller"





categories:
  - devOps
  - monitoring
  - sensu
comments: true
date: 2012-02-27 11:06:38 -0800
date_gmt: 2012-02-27 18:06:38 -0800
published: true
status: publish
tags: []
title: Sensu handler sets
url: /2012/02/27/sensu-handler-sets/


---

In past articles we have covered some of basics of Sensu handlers. A nice feature we haven't touched on yet is handler "sets". Handler sets were added around v0.9.2 and can be quite useful for saving time when modifying your handler.

<!--more-->

For example, consider you have a standard set of handlers that you assign to most of your checks -- pagerduty, irc, campfire. Now, suppose you want to add the [GELF](https://github.com/sonian/sensu-community-plugins/blob/master/handlers/notification/gelf.rb) ( [graylog2](http://graylog2.org)) handler to all of your monitors as well. If each of your checks is defined as such:

{{< highlight javascript >}}
{
  "checks": {
    "all_disk_check": {
      "notification": "Diskspace Too Low",
      "command": "PATH=$PATH:/usr/lib64/nagios/plugins:/usr/lib/nagios/plugins check_disk -w 25% -c 15% /",
      "subscribers": ["all"],
      "interval": 60,
      "handlers": ["pagerduty", "irc", "campfire"]
    }
  }
}
{{< / highlight >}}

... you would need to modify every check's "handlers" attribute to include your new "gelf" handler. If you have a lot of checks this can be a little bit of a burden.

Handler sets save a lot of time in this situation. If you instead define your checks like this:

{{< highlight javascript >}}
{
  "checks": {
    "all_disk_check": {
      "notification": "Diskspace Too Low",
      "command": "PATH=$PATH:/usr/lib64/nagios/plugins:/usr/lib/nagios/plugins check_disk -w 25% -c 15% /",
      "subscribers": ["all"],
      "interval": 60,
      "handlers": ["default"]
    }
  }
}
{{< / highlight >}}

And then define your handlers on your Sensu server like the following. Notice we have converted the 'default' handler to a set.

{{< highlight javascript >}}
"handlers": {
      "default": {
        "type": "set",
        "handlers": ["pagerduty", "irc", "campfire", "gelf"]
      },
      "pagerduty": {
        "type": "pipe", 
        "command": "/etc/sensu/handlers/pagerduty"
      },
      "irc": {
        "type": "pipe",
        "command": "/etc/sensu/handlers/irc"
      },
      "campfire": {
        "type": "pipe",
        "command": "/etc/sensu/handlers/campfire"
      },
      "gelf": {
        "type": "pipe",
        "command": "/etc/sensu/handlers/gelf.rb"
      }   
    },
{{< / highlight >}}

You only need to modify the "default" handler set to add the new "gelf" handler and no need to change any of your check definitions.

It's a simple feature, but it can save a bunch of time. You could also setup a handler set specifically for metrics and another set for notifications. In the case of metrics, you could easily ship metrics to multiple systems - Graphite, Librato, Cube, etc. And adding a new system would be as simple as creating the handler and adding it to the handler set. Happy Sensu'ing.
