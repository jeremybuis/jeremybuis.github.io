---
layout: post
title: "Setting Up a Secure Instance of Express JS (GitHub Repo)"
date: 2015-07-08
---

*Originally published on the [Software Secured blog](https://web.archive.org/web/20211129195211/https://www.softwaresecured.com/author/jbuis/). Mirrored here for posterity.*

In a [previous blog post](/blog/2015/04/08/secure-your-express-application) I mentioned ways to secure your ExpressJS instance. This included both using third party modules and modifications to the default configuration of Express.

The blog post received great feedback, so we decided to create a skeleton that showed how to handle the security concerns addressed. The skeleton is a great starting point for a secure ExpressJS application and this post will cover the details getting started with it and what it covers for you out of the box.

The source code for the skeleton can be found here: [dead-simple-express](https://github.com/jeremybuis/dead-simple-express).

Check out the **secure** branch for all the details.

## Getting Started

The following instructions are done with an OSX machine in mind, so modify accordingly.

Make sure to have mongodb installed:

```bash
brew install mongodb
```

To use:

```bash
git clone https://github.com/jeremybuis/dead-simple-express.git && cd dead-simple && rm -rf .git
npm install
bower install
npm start
```

Navigate to `http://localhost:4000` to view the basic page, keeping in mind it's a starting point project, so things are pretty bare.

## What You Get in the Skeleton

- A rock solid starting point for writing an ExpressJS server side webapp
- Sane defaults which includes express configurations and security focused configuration
- Logical app structure which has a nice separation of concerns between files
- Proper error handling
- Security minded modules to handle issues addressed in last blog post
- Build script for super dev powers using Gulp

## Security Issues It Covers

- Cross-site Request Forgery (CSRF)
- Security headers using helmet
- HPP or HTTP Parameter Pollution
- Content length validation
- Downgraded user privileges
- Secure cookies
- Proper env variable loading
- Removal of x-powered-by header
- Generic cookie name
- User accounts with bcrypt password handling

## Recommended Setup

My preference to set something like this up in production is to put your express server behind a nginx proxy.

The proxy handles SSL termination and routes traffic to your express server. It also handles serving static resources. This way your express app is only handling app specific routes that have business logic attached to them.

This setup allows you to not run the express instance as root as it doesn't need to be bound to a port lower than 1024.

That's all for now folks.
