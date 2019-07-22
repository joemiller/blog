---
author: "Joe Miller"
draft: true





categories:
  - howto
  - linux
comments: true
date: 2013-10-09 18:15:04 -0700
date_gmt: 2013-10-10 01:15:04 -0700
published: true
status: publish
tags: []
title: Dell OpenManage on Fedora 19
url: /2013/10/09/dell-openmanage-on-fedora-19/


---

Here's how to get Dell OpenManage running on Fedora 19. It is a simple process but I could not find any up to date information on how to do it elsewhere.

<!--more-->

This [Dell OpenManage wiki page](http://linux.dell.com/wiki/index.php/Repository/hardware) is the primary source of information for running OMSA on Linux. Go there and run the script installer, or install the yum repo files and RPM signing keys manually.

### yum repo

The installer will probably install the `/etc/yum.repos.d/dell-omsa-repository.repo` with URL's such as:

`mirrorlist=http://linux.dell.com/repo/hardware/latest/mirrors.cgi?osname=f$releasever&basearch=$basearch&native=1&sys_ven_id=$sys_ven_id`

Change the `osname=f$releasevar` param and force it to `el6`:

`mirrorlist=http://linux.dell.com/repo/hardware/latest/mirrors.cgi?osname=el6&basearch=$basearch&native=1&sys_ven_id=$sys_ven_id`

Next, run `yum search srvadmin` and you should see the list of OpenManage packages:

{{< highlight bash >}}
srvadmin-all.x86_64 : Meta package for installing all Server Administrator features, 7.3.0
srvadmin-argtable2.x86_64 : A library for parsing GNU style command line arguments, 7.3.0
srvadmin-base.x86_64 : Meta package for installing the Server Agent, 7.3.0
srvadmin-cm.i386 : OpenManage Inventory Collector, 7.3.0
...
{{< / highlight >}}

Install the ones you need. For the basic CLI tools you can install `srvadmin-omcommon` and `srvadmin-omacore`.

### canary file to force dataeng.service to start

It's not obvious from the error output, unfortunately, but the cli tools `omreport` and `omconfig` will not work until the `dataeng.service` is started, but it will refuse to start when it detects Fedora - and "unsupported" platform.

Trick the startup scripts into attempting to start anyway by creating this file:

{{< highlight bash >}}
touch /opt/dell/srvadmin/lib64/openmanage/IGNORE_GENERATION
{{< / highlight >}}

The presence of this file skips the checks performed in `/opt/dell/srvadmin/sbin/CheckSystemType` during dataeng startup.

### test things

run `sudo /opt/dell/srvadmin/bin/omreport chassis info`, you should see:

{{< highlight bash >}}
Chassis Information


Index : 0
Chassis Name : Main System Chassis
Host Name : hardware_bootstrap_test.pod1.panth.io
iDRAC7 Version : 1.40.40 (Build 17)
Lifecycle Controller 2 Version : 1.1.1.18
Chassis Model : PowerEdge R720
{{< / highlight >}}