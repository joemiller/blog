---
author: "Joe Miller"


draft: true



categories:
  - software
  - internet services
  - AWS
comments: true
date: 2011-01-31 17:34:13 -0800
date_gmt: 2011-02-01 00:34:13 -0800
published: true
status: publish
tags:
  - amazon
  - aws
  - cloudfront
  - private
  - streaming
title: Amazon Cloudfront Private Streaming CLI Tools
url: /2011/01/31/amazon-cloudfront-private-streaming-tools/


---

Back in late 2010 I had a need to create a private Cloudfront streaming distribution but there was no simple way to do this. Amazon's web-based AWS Management Console did not support this and I could not find any simple CLI tools for doing this.

Luckily, RightScale's _right\_aws_ ruby gem (>= 2.0.0) provides support for the Cloudfront private streaming API.

These tools should make it simple for any linux admin or developer to get started with creating a private streaming distribution on Amazon.

<!--more-->

The code and detailed documentation is available on github:
 [https://github.com/joemiller/aws-cf-private-streaming-tools](https://github.com/joemiller/aws-cf-private-streaming-tools)

To read more, and see example usage, click past the break!

### Example Usage

In this example we will setup a new Cloudfront Private Streaming distribution with the following attributes:

- S3 origin bucket: my-video-bucket
- CF base URL (CNAME): rtmp://cf.example.com/

#### 1. Setup AWS keys

{{< highlight text >}}
$ export AWS_ACCESS_KEY_ID='xxxxx'
$ export AAWS_SECRET_ACCESS_KEY='xxxxxx'
{{< / highlight >}}

#### 2. Create a new Cloudfront Streaming Distribution

{{< highlight text >}}
$ ./cf-streaming-distribution.rb create my-video-bucket \
    --cname cf.example.com \
    -m "private streaming distribution (rtmp) with origin bucket: my-video-bucket"


 Success!
 domain_name: s1loj2pirm00it.cloudfront.net
 aws_id: E1UGDLB9XZBD79
{{< / highlight >}}

#### 3. Configure CNAME in your DNS server

This part will depend on DNS server or DNS provider. You'll need to create a new CNAME for cf.example.com --> s1loj2pirm00it.cloudfront.net

#### 4. Create a new Origin-Access-ID (OAI)

{{< highlight text >}}
$ ./cf-origin-access-id.rb create "OAI for use on the cf.example.com distribution"


 Success!
 AWS_ID : E2CWXW7A1B3YIU
 Location : https://cloudfront.amazonaws.com/origin-access-identity/cloudfront/E2CWXW7A1B3YIU
 S3 Canonical ID: 3b5285f7f1b51ff2e63e8ff8127b7ffb76edee24580cb7fff6ef812aa87b749aaa3ed1aab389aaaab4453499a7ba57e7
{{< / highlight >}}

#### 5. Assign the OAI to the Cloudfront distribution

{{< highlight text >}}
$ ./cf-streaming-distribution.rb modify E1UGDLB9XZBD79 --oai E2CWXW7A1B3YIU
 Success!
{{< / highlight >}}

#### 6. Grant the OAI access to the files in the S3 bucket

{{< highlight text >}}
$ ./cf-origin-access-id.rb grant E2CWXW8B1U3YJU my-video-bucket
 Applying grant [E2CWXW8B1U3YJU:'FULL_CONTROL'] on: my-video-bucket/flvs/video01.flv
 Applying grant [E2CWXW8B1U3YJU:'FULL_CONTROL'] on: my-video-bucket/flvs/video02.flv
...
{{< / highlight >}}

#### 7. Create RSA Keypair on the Amazon AWS website

You cannot create keypairs with the cloudfront API, so you'll need to do this step on the AWS website.

- Goto http://aws.amazon.com then login:
- Account > Security Credentials > Key Pairs
- Click “Create New Key Pair” under the “Cloudfront Key Pairs” section
- A keypair will be created and the private key will automatically begin downloading.

You must save this file! it will be in the form “pk-XXXXXX.pem”. If you lose this key, you can’t get it back because Amazon only stores the public key.

#### 8. Register the account and keypairs on the cloudfront distribution

NOTE: the --trusted-signer arguments takes an amazon account ID as an argument.  
The special ‘self’ can be used instead.

{{< highlight text >}}
$ ./cf-streaming-distribution.rb modify E1UGDLB9XZBD79 --trusted-signer self
 Success!
{{< / highlight >}}

#### 9. Verify settings on the new private Streaming Distribution

{{< highlight text >}}
$ ./cf-streaming-distribution.rb get E1UGDLB9XZBD79
 AWS_ID : E1UGDLB9XZBD79
 E_TAG : EQ3HGAPOK1IFN
 Status : InProgress
 Enabled : true
 domain_name : s1loj2pirm00it.cloudfront.net
 origin : my-video-bucket.s3.amazonaws.com
 CNAMEs : cf.example.com
 Comment : private streaming distribution (rtmp) with origin bucket: my-video-bucket
 Origin Access ID: origin-access-identity/cloudfront/E2CWXW7A1B3YIU
 Trusted Signers : self
 Active Signers:
     -> aws_account_number: self
          -> key_pair_id : APDBDOEHALFXGK5AQU5R
{{< / highlight >}}

NOTE: The distribution will not be usable until Status changes from InProgress to Deployed. This can take up to 15minutes.

You can also use the command `cf-streaming-distribution.rb wait AWS_ID` to wait for a distribution to change from InProgress to Deployed. The command will exit as soon as the status changes to Deployed. This is useful for scripts where you need to control timing.
