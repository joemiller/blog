---
author: "Joe Miller"


draft: true



categories:
  - software
  - devOps
comments: true
date: 2010-09-27 14:33:59 -0700
date_gmt: 2010-09-27 21:33:59 -0700
published: true
status: publish
tags:
  - mcollective
  - shellcmd-agent
  - devops
  - capistrano
title: shellcmd-agent - Execute any shell command in parallel on multiple machines with mcollective
url: /2010/09/27/shellcmd-agent-execute-any-shell-command-in-parallel-on-multiple-machines-with-mcollective/


---

Use mc-shellcmd to run any command on the nodes in your collective, eg:

<!--more-->

{{< highlight text >}}
$ mc-shellcmd 'echo I execute therefore I am'
===============================================================
web01.example.com exitcode: 0, output:
I execute therefore I am
===============================================================
web02.example.com exitcode: 0, output:
I execute therefore I am
===============================================================
web03.example.com exitcode: 0, output:
I execute therefore I am
===============================================================
db01.example.com exitcode: 0, output:
I execute therefore I am
===============================================================


Finished processing 4 / 4 hosts in 221.02 ms
{{< / highlight >}}

I have been experimenting recently with [mcollective](http://marionette-collective.org/) in my environment.  If you're not familiar with mcollective it is a _"framework to build server orchestration or parallel job execution systems."_

The simplest way to explain it is that mcollective is "SSH for-loop 2.0" or "next-gen clusterSSH", but that doesn't do enough justice to how powerful mcollective is likely to become.  Check out R.I.Pienaar's automated [mcollective-server-provisioner](http://github.com/ripienaar/mcollective-server-provisioner "mcollective-server-provisioner") to see a great example of mcollective's power and potential to be a revolutionary platform as systems administration moves from the age of managing stable, fixed assets into managing flexible, dynamic building blocks (ie: [devOps](http://www.jedi.be/blog/2010/02/12/what-is-this-devops-thing-anyway/ "devOps").)

I still find myself needing to do ad-hoc commands on many hosts as quickly as possible.  I still use [Capistrano](http://www.capistranorb.com/ "Capistrano") to do most of this, but I also wanted to be able to run arbitrary commands via mcollective.  Thus, shellcmd-agent was born.

BEWARE!  This is a powerful agent.  Anyone with access to your mcollective message bus can run any command as root on your servers.

You can find the code and documentation on github: [http://github.com/joemiller/shellcmd-agent](http://github.com/joemiller/shellcmd-agent "http://github.com/joemiller/shellcmd-agent")
