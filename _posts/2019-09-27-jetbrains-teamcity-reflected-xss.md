---
layout: post
title: "JetBrains TeamCity Reflected XSS"
date: 2019-09-27
---

*Originally published on the [Software Secured blog](https://web.archive.org/web/20230201221817/https://www.softwaresecured.com/jetbrains-teamcity-reflected-xss/). Mirrored here for posterity. CVE-2019-15848.*

## How We Found the TeamCity XSS

On a recent client engagement, we were challenged to gain access to their private CI server. The CI server suffered from a security misconfiguration, and we were able to gain access. This was already a success, but we wanted to show more impact.

We started looking around the client's TeamCity instance to see how we could increase the impact of the issue. While manually crawling the site, with Burp open in the background, Burp popped a DOM-based XSS issue. This is generally a false positive, but we had a look anyway.

This is where the fun began.

The analysis Burp Suite produced was the following:

> Data is read from `location.href` and passed to `jQuery.html`.

The following value was injected into the source:

```
https://teamcity.jetbrains.com/project.html?projectId=ja21ikln4q%27%22`'"/ja21ikln4q/><ja21ikln4q/>eqoajxnqdk&cb_Root&fromExperimentalUI=ja21ikln4q%27%22`'"/ja21ikln4q/><ja21ikln4q/>eqoajxnqdk&true&tab=ja21ikln4q%27%22`'"/ja21ikln4q/><ja21ikln4q/>eqoajxnqdk&stats
```

I tried the PoC, it didn't work, and it returned a 404.

Let us try and get a working URL, and see if any injections are reflected. A working URL looks like:

```
https://teamcity.jetbrains.com/project.html?projectId=cb_Root&fromExperimentalUI=true&tab=stats
```

I tried injecting payloads I could easily search for in the DOM, like `xINJECTx`, into each query string parameter.

For the `tab` parameter, I found that my payload was reflected in the page inside a JavaScript context. This means my `xINJECTx` was inside script tags when the page was rendered. I sent the request and was returned:

```html
<script>
ReactUI.renderConnected('open_in_experimental_ui', ReactUI.OpenInExperimentalUI, {
  tab: 'xINJECTx'
});
</script>
```

This has the potential of a reflected XSS!

I tried the URL `https://teamcity.jetbrains.com/overview.html?tab='-alert(document.cookie)-'` and the payload:

```
'-alert(document.cookie)-'
```

**Jackpot!**

This means our injection is returned to the browser, where the payload executes. This can lead to many outcomes, some of which have high impact, including the potential to steal CSRF tokens, which can lead to account takeover. The injection was found to work on 4 pages in total.

Another researcher found the same XSS independently of us, and was able to show more impact by proving they were able to gain remote command execution.

The issue was also valid on the open source TeamCity instance, so the examples above are all based on that instead of the private client data.

We disclosed the issue privately to JetBrains, and they promptly created a fix. The XSS affects versions 2019.1 and 2019.1.1 of TeamCity and is fixed in version 2019.1.2.

## The Fix

The fixed script snippet looks like:

```html
<script>
ReactUI.renderConnected('open_in_experimental_ui', ReactUI.OpenInExperimentalUI, {
  favoriteBuilds: true,
  tab: ReactUI.queryToObject(location.search).tab
});
</script>
```

This safely accepts input from the tab query string parameter — user input is no longer reflected onto the page.

## How to Fix XSS

Always dig a little deeper to see how we can increase the impact of our findings.

In general, the key to fixing XSS in any form is to apply the proper encoding to the output before being rendered in a browser. In this case, the fix was to remove the reflected payload, and access the query string value in a safe way. This is a better approach in this case, as user input is no longer reflected onto the page.

## Timeline

- **July 16, 2019** – Found reflected XSS issue
- **July 17, 2019** – Reported reflected XSS issue
- **July 18, 2019** – Issue updated to fixed
- **July 19, 2019** – Issue verified as fixed
- **July 31, 2019** – TeamCity 2019.1.2 released
- **September 26, 2019** – Quarterly Security Bulletin released

## References

- [TeamCity 2019.1.2 Release](https://blog.jetbrains.com/teamcity/2019/07/teamcity-2019-1-2-is-released/)
- [TeamCity 2019.1.2 Release Notes](https://confluence.jetbrains.com/display/TW/TeamCity+2019.1.2+%28build+66342%29+Release+Notes)
- [JetBrains Security Bulletin Q2 2019](https://blog.jetbrains.com/blog/2019/09/26/jetbrains-security-bulletin-q2-2019/)
- Independent discovery by [@JLLeitschuh](https://twitter.com/JLLeitschuh/status/1169332316612644864)
