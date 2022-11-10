---
title: Easy SPTPeasy
---

The powers that be here at SHU make some webspace available to faculty
and staff here for course-related materials. This is a nice convenient
alternative to the LMS system for webhosting, and I wanted to explore
what else we could do with it. My contact at the Teaching Learning
Center on campus set me up with some space and an account. That I can
use as soon as I can log in and get to it. All technology and know-how
around campus seems geared toward a pointy-clicky Windows based mode
of operation.

Easy command line access requires a little bit of study and
configuration.

Firstly, the right tool for the job.

## BTW, `SFTP` =/= `FTPS`

If you don’t know, `SFTP` and `FTPS` are not the same thing. If a
person talks to you about "secure FTP", stop that person immediately
and make them clarify to you which they mean. It’s possible that they
don’t know either: trust, but verify.

I found the easy way to connect to our server
(`tltc-web1h-prod.shu.edu`) was to simply `brew install lftp`. Scads
of dependencies I needed to download and update all handled for me;
man alive package managers are great!

But I run into a problem when I actually try to use this to get to my
space now. I can connect, but when I actually try and move around, you
run into the following error.

```
$ lftp -u hemannja,"$PASSWD" ftp://tltc-web1h-prod.shu.edu/hemannja/
cd: Fatal error: Certificate verification: self-signed certificate ...
```

## Certificates and verification

This error message is saying that the server has a self-signed
certificate authority, and so my system rightly doesn't trust it. If
this were a nefarious actor, well, it wouldn't make any sense to take
his own word that he's an honest fella. But I happen to know that I'm
really dealing with the machine I think I am, so I want to, for this
guy only, accept the certificate that it's providing me. 

So I need to _get_ the certificate that this server is presenting me

```
$ openssl s_client -connect tltc-web1h-prod.shu.edu:21 -starttls ftp
...
---
Server certificate
-----BEGIN CERTIFICATE-----
123...
...
...456789abcdefgihijklm==
-----END CERTIFICATE-----
subject=...
issuer=...
---
```

I needed to save that certificate itself as a certificate file. Just
the certificate, mind you.

```
$ cat > ~/tltc_web1h_prod_AES-256.crt
...
```

Once I had that, I needed to add it so that `lftp` knew about it.

Luckily this was a [solved
problem](https://stackoverflow.com/a/65760327/4355474). If you want to
add it to your MacOS Keychain, [they also have a help page for
this](https://support.apple.com/en-my/guide/keychain-access/kyca2431/mac).


