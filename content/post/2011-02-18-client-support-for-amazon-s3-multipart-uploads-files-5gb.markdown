---
author: "Joe Miller"


draft: true


draft: true

categories:
  - software
  - internet services
  - macosx
  - AWS
comments: true
date: 2011-02-18 16:07:22 -0800
date_gmt: 2011-02-18 23:07:22 -0800
published: true
status: publish
tags: []
title: Client support for Amazon S3 multipart uploads (files >5GB)
url: /2011/02/18/client-support-for-amazon-s3-multipart-uploads-files-5gb/


---

I recently had a need to upload a few large (11GB) files to S3 for offsite archive purposes. Amazon announced support for objects >5GB back in [November 2010](http://aws.amazon.com/about-aws/whats-new/2010/11/10/Amazon-S3-Introducing-Multipart-Upload/ "http://aws.amazon.com/about-aws/whats-new/2010/11/10/Amazon-S3-Introducing-Multipart-Upload/"), so I had assumed that most S3 clients and libraries had support for it already.  I was wrong.  It turns out that in order to support large files, you need to use the new "multipart upload" feature.  What I found was that many S3 clients do not yet support multipart uploads.

In case this helps save anyone else a few hours of time, here is a summary of what I found regarding multipart upload support in the various S3 clients:

<!--more-->

As of Feb 18, 2011:

- Multipart uploads supported:
  - [S3 Browser (FREE and PRO) - Win32](http://s3browser.com/ "S3 Browser")
  - [CloudBerry Explorer (PRO only) - Win32](http://cloudberrylab.com/ "CloudBerry Explorer")
  - **UPDATE** :   [Cyberduck 4 - Mac OSX / Win32](http://cyberduck.ch/ "Cyberduck")
  - **UPDATE** : [s3cmd  - python, multi-platform (version 1.1.0-beta2 +)](http://s3tools.org/s3cmd "s3cmd")

- Multipart uploads NOT supported:
  - [Cyberduck - Mac OSX](http://cyberduck.ch/ "Cyberduck")
  - [Transmit - Mac OSX](http://www.panic.com/transmit/ "Transmit FTP client for Mac")
  - [CloudBerry Explorer (FREE version) - Win32](http://cloudberrylab.com/ "CloudBerry")
  - [s3cmd  - python, multi-platform](http://s3tools.org/s3cmd "s3cmd")
  - [Bucket Explorer - multi-platform](http://www.bucketexplorer.com/ "Bucket Explorer")

To solve my immediate need I used S3 Browser on Windows 7.  It was the only _free_ client I found that supported multipart uploads.  I mostly use a Mac, however, so if someone knows of a free S3 client for Mac please let me know.

I did not look at which S3 libraries support multipart uploads yet.  I have heard that my favorite python library - [boto](http://code.google.com/p/boto/ "boto aws library") - supports it already.  And I have seen some reports that RightScale's [right\_aws](http://rightaws.rubyforge.org/ "right\_aws") ruby library may support it on some platforms.  I'm not sure about the others.  Post in the comments if you want to confirm support for multipart uploads in your favorite S3 client or library.

**Update (3/10/2010):** Cyberduck 4.x was released this week and includes support for multipart uploads and 5TB uploads.  A Windows version is also available now.  Pretty cool, good job Cyberduck.

**Update (4/18/2012):** [s3cmd 1.1.0-beta2](http://s3tools.org/s3cmd-110b2-released "s3cmd 1.1.0-beta2 release announcement") is available and supports multipart uploads along with many other new S3 and Cloudfront features. Grab the beta or wait for 1.1.0 final to be released
