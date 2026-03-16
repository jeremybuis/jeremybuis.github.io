---
layout: post
title: "Elementor Page Builder 2.9.8 Stored XSS"
date: 2020-06-05
---

*Originally published on the [Software Secured blog](https://web.archive.org/web/20221130005129/https://www.softwaresecured.com/elementor-page-builder-stored-xss/). Mirrored here for posterity. CVE-2020-13864 and CVE-2020-13865.*

## Introduction

At Software Secured, we routinely test our own infrastructure to make sure we practice what we preach.

The Software Secured website is powered by WordPress. During a routine test of our website, we decided to review our installed WordPress plugins and the existence of public vulnerabilities for those plugins. We listed our plugins, checked our versions and found they were all up to date without vulnerabilities. We decided to dig deeper.

Elementor caught our attention as it's a powerful plugin with an excellent feature set which provided a large scope for us to test. There have been [interesting vulnerabilities](https://wpvulndb.com/plugins/elementor) in the Elementor editor in the past — most consisting of cross-site scripting (XSS) vulnerabilities.

## XSS #1

### Discovery

We created a draft post and opened it in the Elementor editor. Elementor uses a drag-and-drop UI, where users pick UI widgets that they would like in their post. We started by looking at an image widget.

The Elementor image widget allows the use of a custom link, so that when a user clicks the image, they are navigated elsewhere on the site. This has potential for XSS!

### Exploitation

Custom links should ring alarm bells for security testers as links with the `javascript:` protocol are susceptible to XSS. Browsers will execute arbitrary JavaScript when used with a `javascript:` protocol:

```
javascript:alert('XSS')
```

We placed this into the custom link field and previewed the page. When the image is clicked, the XSS payload fires.

The above proof-of-concept was viable for **all UI widgets** that allowed the use of a custom URL.

## XSS #2

### Discovery

We continued testing with the image widget, and jumped to the advanced tab, as this is usually a good spot to look for extra configuration.

The advanced tab has an option for **custom attributes**, which has XSS written all over it.

Our first test looked like:

```
xss|xss
onload|alert()
onclick|alert()
onmouseover|alert()
```

As we didn't know what we were dealing with yet. The `xss|xss` row was added as an attribute, but the rest were all stripped. How do we find the correct HTML attributes that will be left unsanitized?

In comes the ever valuable [PortSwigger XSS cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet).

### Exploitation

We found a payload that affects all browsers and works on `div` elements. The trouble was that it required adding an animation style to the page and applying the style to our `div` element. Of course, Elementor has a solution for adding custom CSS to an element, and we could include the extra style attribute the same way we were including our other attributes.

We previewed our page and **success!**

This time, our attributes were not stripped or modified by the server, and our payload is returned to the browser, where it executes when the page loads.

The above proof-of-concept was viable for **all UI widgets** that Elementor supports, as they all support custom CSS and attributes.

We promptly wrote up both issues, did a happy dance, and then privately disclosed the issue to the Elementor security team. We applied for a CVE to help the community track the above two vulnerabilities, and were given **CVE-2020-13864**.

## Bypass for XSS #1

Once XSS #1 was fixed by the Elementor team, we re-opened our PoC draft blog post to validate the fix.

By utilizing the PortSwigger cheat sheet again, we were able to find a bypass for the initial fix. We reported our bypass and waited for a fix. The bypass was given **CVE-2020-13865**.

## Impact

A low privileged author user is able to launch XSS attacks against admin and unauthenticated victims, which can potentially lead to privilege escalation or malware delivery.

## Recommendations

Update to the latest version of the Elementor plugin.

## Timeline

- **May 20, 2020** – Bug found and private disclosure to Elementor security team
- **May 21, 2020** – Elementor acknowledge receipt of the disclosure
- **May 24, 2020** – Elementor fixed the issue in version 2.9.9
- **May 28, 2020** – Bypass submitted for the above fix
- **May 28, 2020** – Elementor acknowledge receipt of the bypass
- **June 1, 2020** – Elementor fixed the bypass in version 2.9.10
- **June 5, 2020** – CVE-2020-13864 and CVE-2020-13865 provisioned
- **June 5, 2020** – Blog post released

## References

- [WPScan Vulnerability Database](https://wpscan.com/vulnerability/10256)
- [Elementor Changelog](https://elementor.com/pro/changelog/)
- [PortSwigger XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
- [CVE-2020-13864](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-13864)
- [CVE-2020-13865](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-13865)
