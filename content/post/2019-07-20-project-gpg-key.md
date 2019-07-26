---
author: "Joe Miller"
title: "Signing releases with a project GPG key"
date: 2019-07-20T07:50:48-07:00
draft: true
---

Intro
=====

This article presents a step-by-step guide for signing a project's releases with GPG. It came from my
own experience of adding GPG-signing support to [vault-token-helper](https://github.com/joemiller/vault-token-helper).

Why sign releases? The Apache Foundation has an guide
[rationale](https://www.apache.org/dev/release-signing.html) already so I'll just
borrow their rationale:

> A signature allows anyone to verify that a file is identical to the one created by
> the Apache release manager. Using a signature:
>
> * users can make sure that what they received has not been modified in any way, either accidentally via a faulty transmission channel, or intentionally (with or without malicious intent)
>
> * the Apache infrastructure team can verify the identity of a file
>
> OpenPGP signatures confer the usual advantages of digital signatures: authentication, integrity and non-repudiation. MD5 and SHA checksums only provide the integrity part as they are not encrypted
>

If your project happens to be lucky enough to use [goreleaser](https://goreleaser.com/) you
can benefit from its built-in support for [GPG signing](https://goreleaser.com/customization/#Signing).  If you don't use goreleaser it is still fairly straightforward to sign releases.

My initial thought was to create a sub-key for signing a project's releases and then I
came across this 

Create the project master GPG key
---------------------------------

This guide assumes you have already created a personal GPG key. This is referred to as an "author key"
in this guide and will be used to sign the project key, verifying your relationship to the project.

List current secret keys (`-K /--list-secret-keys`):

```console
$ gpg --keyid-format long -K

/Users/joe/.gnupg/pubring.kbx
-----------------------------
sec   rsa4096/ADC517A11EB0B309 2016-02-09 [SC]
      Key fingerprint = CF5C 5827 E7BD 7059 7992  8EC4 ADC5 17A1 1EB0 B309
uid                 [ultimate] Joe Miller <joemiller@keybase.io>
ssb   rsa2048/3C7B936B8FC96571 2016-02-09 [E] [expires: 2024-02-07]
ssb   rsa2048/B0FBB6EC7A273832 2016-02-09 [SA] [expires: 2024-02-07]
ssb   rsa2048/4AE3F48180C87370 2019-07-12 [S]
```

Generate a new master GPG key. This will be the "project key":

```console
$ gpg --expert --full-generate-key

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
   (9) ECC and ECC
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (13) Existing key
Your selection? 8

Possible actions for a RSA key: Sign Certify Encrypt Authenticate
Current allowed actions: Sign Certify Encrypt

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? E

Possible actions for a RSA key: Sign Certify Encrypt Authenticate
Current allowed actions: Sign Certify

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? S

Possible actions for a RSA key: Sign Certify Encrypt Authenticate
Current allowed actions: Certify

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? Q
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0
Key does not expire at all
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: vault-token-helper
Email address: vault-token-helper@joemiller.me
Comment: github.com/joemiller/vault-token-helper project key
You selected this USER-ID:
    "vault-token-helper (github.com/joemiller/vault-token-helper project key) <vault-token-helper@joemiller.me>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: key 37F9D1272278CD32 marked as ultimately trusted
public and secret key created and signed.

pub   rsa4096 2019-07-13 [C]
      5EF225507053ACC2728AA51C37F9D1272278CD32
uid        vault-token-helper (github.com/joemiller/vault-token-helper project key) <vault-token-helper@joemiller.me>
```

We now have a new master key with only the `[C]` (certify) bit set. It is a special kind of key
that can only sign sub-keys. We will create a signing sub-key for signing the releases next and
sign it with the master key.

Export the keyid:

```console
export KEYID=5EF225507053ACC2728AA51C37F9D1272278CD32
```

Create a signing sub-key
------------------------

Now that we have established a new master key for the project we will create a sub-key that
will be used for signing releases. By creating a sub-key for signing we are able to keep the
master key safely offline. The sub-key can be deployed to your CI/CD pipeline. If it is compromised
it can be revoked by the master key and replaced with a new sub-key.

Edit the master key:

```console
gpg --expert --edit-key $KEYID
```

Create a signing key by selecting `(4) RSA (sign only):`

```console
gpg> addkey

Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (12) ECC (encrypt only)
  (13) Existing key
Your selection? 4
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0
Key does not expire at all
Is this correct? (y/N) y
Really create? (y/N) y
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

sec  rsa4096/37F9D1272278CD32
     created: 2019-07-13  expires: never       usage: C
     trust: ultimate      validity: ultimate
ssb  rsa4096/6720A9FD78AC13F5
     created: 2019-07-13  expires: never       usage: S
[ultimate] (1). vault-token-helper (github.com/joemiller/vault-token-helper project key) <vault-token-helper@joemiller.me>
```

Save and exit:

```console
gpg> save
```

List secret keys. There are now two keys, our author key and a new project key:

```console
$ gpg --keyid-format long -K

sec   rsa4096/ADC517A11EB0B309 2016-02-09 [SC]
      Key fingerprint = CF5C 5827 E7BD 7059 7992  8EC4 ADC5 17A1 1EB0 B309
uid          [ultimate] Joe Miller <joemiller@keybase.io>
ssb   rsa2048/3C7B936B8FC96571 2016-02-09 [E] [expires: 2024-02-07]
ssb   rsa2048/B0FBB6EC7A273832 2016-02-09 [SA] [expires: 2024-02-07]
ssb   rsa2048/4AE3F48180C87370 2019-07-12 [S]

sec   rsa4096/37F9D1272278CD32 2019-07-13 [C]
      5EF225507053ACC2728AA51C37F9D1272278CD32
uid         [ultimate] vault-token-helper (github.com/joemiller/vault-token-helper project key) <vault-token-helper@joemiller.me>
ssb   rsa4096/6720A9FD78AC13F5 2019-07-13 [S]
```

Sign the project key with your author key:

```console
gpg --sign-key $KEYID
```

Publish the project key to the key server network. This allows others to easily download
the project's public key to verify releases:

```console
gpg --send-key $KEYID
```

Prepare keys for CI/CD
----------------------

Now that we have created our project key and a sub-key for signing we will export the
sub-key and base64 encode it for use in our CI/CD system.

List the master and sub-key ID's:

```console
$ gpg --keyid-format long -K $KEYID

sec   rsa4096/37F9D1272278CD32 2019-07-13 [C]
      Key fingerprint = 5EF2 2550 7053 ACC2 728A  A51C 37F9 D127 2278 CD32
uid            [ultimate] vault-token-helper (github.com/joemiller/vault-token-helper project key) <vault-token-helper@joemiller.me>
ssb   rsa4096/6720A9FD78AC13F5 2019-07-13 [S]
```

Export the sub-key ID:

```console
export SUBKEY=6720A9FD78AC13F5
```

Export the sub-key - and **ONLY** the sub-key. The `!` is important for exporting only the sub-key:

```console
gpg --armor --export-secret-subkeys $SUBKEY\! >vault-token-helper.signing-key.gpg
```

Base64 encode the signing-key and copy it into your clipboard. `pbcopy` copies stdin to the clipboard on macOS:

```console
cat vault-token-helper.signing-key.gpg | base64 | pbcopy
```

Create an environment variable `GPG_KEY` in your CI system using the base64 encoded copy of the signing key as the contents.

Take the master key offline
---------------------------

This step is optional but it increases the security of the project master key. In this stage
we will delete the master key from our keychain and leave only the signing sub-key. This will
allow us to sign builds locally. You could also delete the entire master key and sub-key if you
wish.

Make a backup of the master key and sub-key. This should be saved in a safe offline space
such as an encrypted USB key.

```console
gpg --armor --export-secret-keys $KEYID  >vault-token-helper.gpg
```

Delete the master key from your keychain:

```console
gpg --delete-secret-keys $KEYID
```

In the future you will need access to the master key such as to add new sub-keys, revoke
existing sub-keys, etc. To do so import the master key from the backup:

```console
gpg --import vault-token-helper.gpg
```

Test sign with goreleaser
-------------------------

*TODO: need to do something better with this section.. maybe show circleci example.. or at least*
   *show running goreleaser config snippet and running without make file*

Testing the release and sign process:

```console
$ GPG_KEY="$(cat vault-token-helper.signing-key.gpg | base64)" make snapshot

$ ll dist/*checksum*

-r--r--r--  1 joe  staff   943B Jul 13 17:03 dist/vault-token-helper_SNAPSHOT-64a3f96_checksums.txt
-rw-r--r--  1 joe  staff   566B Jul 13 17:03 dist/vault-token-helper_SNAPSHOT-64a3f96_checksums.txt.sig

$ gpg --verify dist/vault-token-helper_SNAPSHOT-64a3f96_checksums.txt.sig dist/vault-token-helper_SNAPSHOT-64a3f96_checksums.txt

gpg: Signature made Sat Jul 13 17:03:05 2019 PDT
gpg:                using RSA key F2B849E45FBA0CBFA0ECC9F36720A9FD78AC13F5
gpg: Good signature from "vault-token-helper (github.com/joemiller/vault-token-helper project key) <vault-token-helper@joemiller.me>" [ultimate]
Primary key fingerprint: 5EF2 2550 7053 ACC2 728A  A51C 37F9 D127 2278 CD32
     Subkey fingerprint: F2B8 49E4 5FBA 0CBF A0EC  C9F3 6720 A9FD 78AC 13F5
```
