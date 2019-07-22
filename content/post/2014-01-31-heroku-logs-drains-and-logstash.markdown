---
author: "Joe Miller"
tags:
  - linux
  - monitoring
  - logging
  - heroku
  - logstash
comments: true
date: 2014-01-31 11:45:58 -0800
draft: false
title: Heroku log drains into Logstash
---

The first and obvious option for shipping logs from a heroku app to Logstash is the [heroku input plugin](http://logstash.net/docs/1.3.3/inputs/heroku). However, this requires installing the Heroku gem and deploying the login + password of a Heroku user to your Logstash server(s). At this time it seems that any user given permissions to an app on Heroku has full control. Not good when you just want to fetch logs. Heroku has added more granular permissions via OAuth but the Heroku gem does not support OAuth tokens yet.

Fortunately there's another option using Heroku's log drain. Setting up a log drain from Heroku to Logstash is almost as simple as the Heroku input plugin but has the major advantage of not requiring any new users or passwords to be deployed on the Logstash server.

<!--more-->

Hooking up your Heroku-deployed apps to your Logstash/Kibana/Elasticsearch infrastructure is straightforward using the syslog drains provided by Heroku. Here is a recipe for putting the pieces together:

# Install Heroku toolbelt

In order to configure a log drain on Heroku you need to install the Heroku toolbelt (or using the API directly). At this time I don't think there's a way to configure log drains from the web UI.

The heroku toolbelt can also be used to tail an apps' logs on the command line which is great for debugging since the logs output by this command are identical to the logs that should be sent to the Logstash if everything is configured correctly:

List apps:

{{< highlight console >}}
$ heroku apps

myapp1 email@dom.tld
myapp2 email@dom.tld
{{< / highlight >}}

Tail app's logs:

{{< highlight console >}}
$ heroku logs --app myapp1 -t

2014-01-31T14:26:11.801629+00:00 app[web.1]: 2014-01-31T14:26:11.801Z - debug: FETCHING TICKET: 15846
2014-01-31T14:26:26.753977+00:00 app[web.1]: 2014-01-31T14:26:26.752Z - debug: FETCHING TICKET: 15851
2014-01-31T14:26:27.457415+00:00 heroku[router]: at=info method=GET path=/ping? host=myapp1 request_id=6cb9b9eb-388c-4364-9278-a81179067f21 fwd="50.57.38.1" dyno=web.2 connect=7ms service=6ms status=200 bytes=2
{{< / highlight >}}

# Configure log drain

Use the Heroku toolbelt to configure a log drain:

{{< highlight console >}}
$ heroku drains:add --app myapp1 syslog://logstash.dom.tld:1514
{{< / highlight >}}

Heroku's Logplex system will now send all logs generated from `myapp1` to `logstash.dom.tld:1514` via TCP in syslog RFC-5424 format.

# Configure Logstash

Next, configure a Logstash input to receive these logs:

{{< highlight console >}}
input {
    tcp {
        port => "1514"
        tags => ["input_heroku_syslog"]
    }
}
{{< / highlight >}}

Heroku uses the syslog format as defined in RFC 5424. Logstash ships with a grok rules that parse out most syslog formats including RFC 5424 but I found that they were not quite perfect for Heroku logs.

For more details on Heroku log drains: [https://devcenter.heroku.com/articles/logging](https://devcenter.heroku.com/articles/logging)

Here are examples of raw log lines sent by Heroku. These are exactly what Logstash will receive and suitable for testing with the [grok debugger](http://grokdebug.herokuapp.com/).

The first is an example of a log message sent from a heroku component, in this case the Heroku router:

{{< highlight console >}}
231 1 2014-01-08T01:05:27.967180+00:00 d.9bc44987-ff40-40ac-a248-ff4ec4d71d7c heroku router - - at=info method=POST path=/ host=myapp1.herokuapp.com fwd="204.236.185.9" dyno=web.1 connect=2ms service=81ms status=200 bytes=2
{{< / highlight >}}

Next, is an example of a log line generated from the application's stdout:

{{< highlight console >}}
158 1 2014-01-08T17:49:14.822585+00:00 d.9bc44987-ff40-40ac-a248-ff4ec4d71d7c app web.1 - - 2014-01-08T17:49:14.822Z -minfo: FETCHING TICKET: 15103
{{< / highlight >}}

Here is the grok pattern we use to parse Heroku's syslog RFC 5424-(ish) log messages:

{{< highlight console >}}
filter {
  if "input_heroku_syslog" in [tags] {
    grok {
      match => ["message", "%{SYSLOG5424PRI}%{NONNEGINT:syslog5424_ver} +(?:%{TIMESTAMP_ISO8601:timestamp}|-) +(?:%{HOSTNAME:heroku_drain_id}|-) +(?:%{WORD:heroku_source}|-) +(?:%{DATA:heroku_dyno}|-) +(?:%{WORD:syslog5424_msgid}|-) +(?:%{SYSLOG5424SD:syslog5424_sd}|-|) +%{GREEDYDATA:heroku_message}"]
    }
    mutate { rename => ["heroku_message", "message"] }
    kv { source => "message" }
    syslog_pri { syslog_pri_field_name => "syslog5424_pri" }
  }
}
{{< / highlight >}}

A few notes about this filter config:

- The log message will be stored in the `message` field.
- Key/value pairs matching `key=value` in the `message` will be parsed and added to the Logstash event. Many of the internal Heroku components's logs include useful key/vals.
- Special fields `heroku_drain_id`, `heroku_source`, and `heroku_dyno` will be extracted from each event.
