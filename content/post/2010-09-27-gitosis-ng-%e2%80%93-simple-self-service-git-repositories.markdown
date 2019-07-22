---
author: "Joe Miller"



draft: true


categories:
  - software
  - git
  - gitosis-ng
comments: true
date: 2010-09-27 15:46:37 -0700
date_gmt: 2010-09-27 22:46:37 -0700
published: true
status: publish
tags:
  - gitosis-ng
  - git
title: gitosis-ng – Simple, self-service git repositories
url: /2010/09/27/gitosis-ng-%e2%80%93-simple-self-service-git-repositories/


---

Gitosis-ng -- it's gitosis with some new features to help users work with the git server. Mainly implemented with commands sent via ssh, ex:

<!--more-->

list available commands:

{{< highlight text >}}
$ ssh git@git.example.com help


    add-repo : Create new repository. Usage: 'add-repo repo_name'
    help : Get list of available commands
    list : Get detailed list of git repositories
    list-json : Get detailed list of repositories in JSON format
    list-short : Get simple list of git repositories
    list-yaml : Get detailed list of repositories in YAML format
{{< / highlight >}}

List available repos:

{{< highlight text >}}
$ ssh git@git.example.com list
    Repository : myproject.git
     Initialized: no
     Your Access: read/write
     Owner : jmiller
     URL :
     Description:


     Repository : otherproject.git
     Initialized: no
     Your Access: readonly
     Owner : bsmith
     URL :
     Description:
{{< / highlight >}}

Create new repo:

{{< highlight text >}}
$ ssh git@git.example.com add-repo myproject


    Created repository: myproject


    Next Steps:
      mkdir myproject
      cd myproject
      git init
      touch README
      git add README
      git commit -a -m 'first commit'
      git remote add origin git@GIT_SERVER_ADDRESS:myproject.git
      git push origin master


    Existing Git repo?
      cd myproject
      git remote add origin git@GIT_SERVER_ADDRESS:myproject.git
      git push origin master
{{< / highlight >}}

I am a fan [github.com](http://github.com "github.com") and [gitorious.com](http://gitorious.com "gitorious.com"), but we needed something internal that would be simpler than gitorious to get up and running.

The next best thing I could find was [gitosis](http://scie.nti.st/2007/11/14/hosting-git-repositories-the-easy-and-secure-way) which is very simple and straightforward.

However, gitosis lacked the user-friendly features I liked about github/gitorious. For example, how about a real simple way to list repos?

So, I decided to add "self-service" features to gitosis and call it gitosis-ng.  I want my users to be able to do all of these tasks on their own:

- List available repos
- Create new repos
- Give permissions to other users on repos they control
- Set metadata on repos (eg: URL, owner contact info, description)

So far I have implemented the first 2 features in this list and will be working on the remaining features as time permits.

Eventually I would love to find a way to allow users to self-register with gitosis-ng, but this could be challenging since gitosis-ng works entirely over SSH and relies on users having an SSH key registered with the server before they can execute commands.

gitosis-ng is available on github:   [http://github.com/joemiller/gitosis-ng](http://github.com/joemiller/gitosis-ng "http://github.com/joemiller/gitosis-ng")
