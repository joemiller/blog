---
author: "Joe Miller"





categories:
  - testing
  - linux
  - sensu
  - vagrant
comments: true
date: 2012-04-26 08:15:40 -0700
date_gmt: 2012-04-26 15:15:40 -0700
published: true
status: publish
tags: []
title: Speeding up Vagrant with parallel provisioning
url: /2012/04/26/speeding-up-vagrant-with-parallel-provisioning/


---

Simple hack for speeding up vagrant provisions.
<!--more-->

Vagrant is an amazing tool. It's quite substantially changed my workflows in a variety of areas. It's a particularly interesting tool for building packages or running tests across multiple OS's or distributions from a single set of scripts.

A recent example of the usefulness of Vagrant is the new packaging and testing work undertaken in the [Sensu](https://github.com/sensu/sensu) project. The project set out to build a new set of native OS packages with a goal of making Sensu easy to deploy on a variety of platforms and without a lot of the friction that sometimes accompanies Ruby apps. As part of the packaging effort we needed a simple mechanism to build native packages on the relevant platforms, ie: .deb's on debian and .rpm on redhat/centos.

We ended up using a combination of Vagrant and some homegrown tools such as [Bunchr](https://github.com/joemiller/bunchr).

You can see the work in these 2 repos:

- [http://github.com/sensu/sensu-build](http://github.com/sensu/sensu-build)
- [http://github.com/joemiller/sensu-tests](http://github.com/joemiller/sensu-tests)

Both codebases contain a `para-vagrant.sh` script that is used in place of the normal `vagrant up` to kick off parallel provisioning tasks. `sensu-tests` is the more interesting example as it runs a set of rspec tests against Sensu across _14_ VM's and this will likely grow to encompass other OS's in the future. The tests are executed as Vagrant provisioners (a combo of Chef and shell to call rspec).

The simplest way to use multi-VM's with Vagrant is the typical `vagrant up`. However, this will boot and run the provisioning tasks sequentially on each VM. With 14 VM's to test, this process can take a long time.

Can we speed this up? Yes. In fact we were able to reduce the time taken to run the `sensu-build` tasks from about 33 minutes to 12 minutes, and reduced `sensu-tasks` from almost 90 minutes to 15!

Here was the first attempt at a parallelization script:

{{< highlight bash >}}
#!/bin/sh


# concurrency is hard, let's have a beer


MAX_PROCS=4


parallel_provision() {
    while read box; do
        echo "Provisioning '$box'. Output will be in: $box.out.txt" 1>&2
        echo $box
    done | xargs -P $MAX_PROCS -I"BOXNAME" \
        sh -c 'vagrant provision BOXNAME >BOXNAME.out.txt 2>&1 || echo "Error Occurred: BOXNAME"'
}


## -- main -- ##


# start boxes sequentially to avoid vbox explosions
vagrant up --no-provision


# but run provision tasks in parallel
cat 
There are 2 steps to the process:
{{< / highlight >}}

- 

Sequentially boot each VM listed in the Vagrantfile, but without running provisioners (`vagrant up --no-provision`). The reason we do the bootups sequentially is to avoid potential kernel panics that VirtualBox is prone to do, at least on OSX.

- 

Feed a list of VM's into `xargs` to execute $MAX\_PROCS `vagrant provision $BOXNAME` processes in parallel. `xargs` will manage the concurrency for us.

The final script we used with the sensu-tests repo is a little different than the above example, which can be viewed here: [para-vagrant.sh](https://github.com/joemiller/sensu-tests/blob/master/para-vagrant.sh). This script reduced the time it takes to run the full test suite from over 90 minutes to 15 minutes. The differences in this final version of the script:

- 

uses GNU parallel instead of xargs for better handling/grouping of the output from the `vagrant provision` processes.

- 

reads the list of VM's from a JSON file (the Vagrantfile also uses this JSON file). To add/remove new VM's, you only need to edit one file now.

### Future?

It should be possible to make the process go even faster by parallelizing the bootup phase, but I have not tried this yet because I'm worried about VirtualBox stability.

Also, I'm sure there's a more polished script/utility that can be written to parallelize any multi-VM Vagrantfile.

### Alternatives?

As [@vvuksan](https://twitter.com/vvuksan) correctly points out, doing such short tasks with Vagrant is probably always going to be a little slow due to the overhead in spinning up VM's, so a possible alternative maybe LXC. There's already a Vagrant-like tool for LXC called [toft](https://github.com/exceedhl/toft). LXC should work fine for workflows that include only various Linux distributions, but won't help if you need to test other OS's like FreeBSD or Solaris.
