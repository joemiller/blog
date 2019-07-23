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
comments: true
date: 2012-01-24 19:22:17 -0800
date_gmt: 2012-01-25 02:22:17 -0800
status: publish
tags: []
title: Re-use Nagios plugins in Sensu for quick profit
url: /2012/01/24/re-use-nagios-plugins-in-sensu-for-quick-profit/


---

In my previous [article](http://joemiller.me/2012/01/19/getting-started-with-the-sensu-monitoring-framework/) I mentioned a key strength of [Sensu](https://github.com/sonian/sensu) is the ability to re-use existing Nagios plugins. This is a powerful feature of Sensu. Nagios has been around for at least 1000 years according to most recent archaeological discoveries, which means a vast amount of human effort (and capital) has gone into creating Nagios plugins. Being able to leverage this prior effort is a [huge](http://www.youtube.com/watch?v=gS7KXhK3sro) win. In this article I'll demonstrate creating a Sensu check with the [check\_http](http://nagiosplugins.org/man/check_http) Nagios plugin.
<!--more-->

## Install the check\_http nagios plugin

Following on the previous article, we'll be doing this demo based off of CentOS 5 but it should not be difficult to find a build of check\_http for your distribution.

First, Make sure you have the EPEL yum repo installed on your machine. Next, let's install the `nagios-plugin-http` package onto our Sensu client node(s):

{{< highlight bash >}}
sudo yum -y install nagios-plugins-http
{{< / highlight >}}

Let's see where the `check_http` binary was installed (note the arch specific path):

{{< highlight bash >}}
$ rpm -ql nagios-plugins-http
/usr/lib64/nagios/plugins/check_http
{{< / highlight >}}

## Write the Sensu check definition

Ok, now we are ready to create a check definition for Sensu. We will create this file on the nodes running sensu-client as well as sensu-server (pro tip: this part is easier if you're using a CM tool like Chef or Puppet.)

`/etc/sensu/conf.d/check_google.json`:

{{< highlight json >}}
{
  "checks": {
    "check_google": {
      "notification": "Google HTTP failed",
      "command": "PATH=$PATH:/usr/lib64/nagios/plugins:/usr/lib/nagios/plugins check_http google.com -R 'search'",
      "subscribers": ["webservers"],
      "interval": 60,
      "handlers": ["default", "pagerduty"]
    }
  }
}
{{< / highlight >}}

Let's look at each part of this check definition:

- `notification`: This can be thought of as a "friendly message". It's most useful with handlers like Twitter or Pagerduty. For example, this will be what you hear when the creepy Pagerduty.com computer voice wakes you up at 3 AM. The full output from the check is also available to handlers in the `output` attribute.
- `command`: This is where we specify the `check_http` command we want to run and relevant options. Notice that we are adding both `/usr/lib64/...` and `/usr/lib/...` to the PATH. This is not required, but it is a nice way to make this check work on both i386 and x86\_64 platforms for the EPEL-supplied version of `check_http`.
- `subscribers`: This is where we specify which sensu-client nodes should run this check. Recall that we defined `"subscriptions": "webservers"` in our `/etc/sensu/conf.d/client.json` in the previous [article](http://joemiller.me/2012/01/19/getting-started-with-the-sensu-monitoring-framework/).
- `interval`: How often this check should be executed.
- `handlers`: The handlers that should receive the output from this plugin. The handlers will be executed on the sensu-server node.

## Turn a profit

After we've created the json file on the clients and servers, we need to restart the sensu-client and sensu-server services so they pickup the new .json file.

In a couple minutes, you should see the following in `/var/log/sensu/sensu-server.log`:

{{< highlight bash >}}
I, [2012-01-23T19:14:22.218301 #7397] INFO -- : [publisher] -- publishing check request -- check_google -- webservers {"message":"[publisher] -- publishing check request -- check_google -- webservers","level":"info","timestamp":"2012-01-23T19:14:22. %6N-0700"}
{{< / highlight >}}

And on the client, in `/var/log/sensu/sensu-client.log`:

{{< highlight bash >}}
I, [2012-01-23T19:15:22.224437 #18787] INFO -- : [subscribe] -- received check request -- check_google {"message":"[subscribe] -- received check request -- check_google","level":"info","timestamp":"2012-01-23T19:15:22. %6N-0700"}
I, [2012-01-23T19:15:22.348884 #18787] INFO -- : [result] -- publishing check result -- check_google -- 0 -- HTTP OK: HTTP/1.0 200 OK - 11294 bytes in 0.099 second response time |time=0.099463s;;;0.000000 size=11294B;;;0
{{< / highlight >}}

There we go. We have implemented a Sensu check using a Nagios plugin. There are quite a few [Nagios plugins](http://nagiosplugins.org/) out there and we should be able to use most (all?) of them with Sensu. Go forth and profit.

For questions on using Sensu, stop by the #sensu channel on Freenode.
