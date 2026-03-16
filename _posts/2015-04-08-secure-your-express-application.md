---
layout: post
title: "Secure Your Express Application"
date: 2015-04-08
---

*Originally published on the [Software Secured blog](https://web.archive.org/web/20211129195211/https://www.softwaresecured.com/author/jbuis/). Mirrored here for posterity.*

## What is Express JS?

At Software Secured, we have been building our internal tools around Node.js and Express. Node.js is becoming more and more popular nowadays and several frameworks have popped up to wrap Node.js functionality and APIs. One of these frameworks is Express JS. Express is very popular today as it was one of the first frameworks for Node.js that handled server-side web tasks. It takes care of mundane tasks, like routing, parameter parsing, templates, and sending responses to the client's browser.

Node.js makes it easier to develop secure applications using Express JS; there are a bunch of security features that can be enabled easily by making simple modifications. The following contains a quick rundown of these features.

## Enable CSRF Protection

This is super easy to setup. We use the `lusca` module:

```javascript
var express = require('express');
var session = require('express-session');
var app = express();
var csrf = require('lusca').csrf();

app.use(lusca.csrf());
```

This is then used in the forms that post to the server:

```html
form(role="form" action="..." method="post")
  input(type='hidden' name='_csrf' value=_csrf)
```

## Don't Run as Root

It's been long foretold by the ancient bearded ops that one shall run a service with the least amount of privilege necessary and no more.

One way to approach this is to drop process privileges after you bind to the port:

```javascript
var express = require('express');
var app = express();

app.listen(app.get('port'), function() {
  process.setgid('www-data');
  process.setuid('www-data');
});
```

Another way is putting something like nginx or another proxy in front of your application. Whatever you do, just don't run as root.

## Turn off X-Powered-By

Why? Because you don't want to make it easy for an attacker to figure what you are running. The `X-Powered-By` header can be extremely useful to an attacker for building a site's risk profile.

```javascript
var express = require('express');
var app = express();

app.disable('x-powered-by');
```

## Use Generic Cookie Name

One way to fingerprint your application is using the cookie name:
- `jsessionid` → Java application
- `phpsessid` → PHP application
- `ASP.NET_SessionId` → .NET application

```javascript
app.use(express.session({
  secret: 'some secret that no one knows',
  key: 'sessionId'
}));
```

## Add HTTPOnly and Secure Flags to Session Cookies

Session cookies should have the SECURE and HTTPOnly flags set. This ensures they can only be sent over HTTPS and there is no script access to the cookie client side.

```javascript
app.use(express.session({
  secret: "s3Cur3",
  key: "sessionId",
  cookie: {
    httpOnly: true,
    secure: true
  }
}));
```

## Use the Content Security Policy (CSP) Header

Using the `helmet` module:

```javascript
var express = require('express');
var helmet = require('helmet');
var app = express();

app.use(helmet.csp({
  defaultSrc: ["'self'"],
  scriptSrc: ["'self'"],
  styleSrc: ["'self'"],
  imgSrc: ["'self'"],
  connectSrc: ["'self'"],
  fontSrc: ["'self'"],
  objectSrc: ["'none'"],
  mediaSrc: ["'self'"],
  frameSrc: ["'none'"]
}));
```

## Use Helmet for Security Headers

Helmet helps set various HTTP headers for security:

```javascript
app.use(helmet.xssFilter());
app.use(helmet.frameguard('deny'));
app.use(helmet.hsts({ maxAge: 7776000000 }));
app.use(helmet.hidePoweredBy());
app.use(helmet.ieNoOpen());
app.use(helmet.noSniff());
app.use(helmet.noCache());
```

## Prevent HTTP Parameter Pollution

Use the `hpp` module to protect against HTTP Parameter Pollution attacks:

```javascript
var hpp = require('hpp');
app.use(hpp());
```

## Validate Content Length

Validate the content length to prevent large payload attacks:

```javascript
app.use(function(req, res, next) {
  if (req.headers['content-length'] > 1E6) {
    return res.status(413).end();
  }
  next();
});
```
