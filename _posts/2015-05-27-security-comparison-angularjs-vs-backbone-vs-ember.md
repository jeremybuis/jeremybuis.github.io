---
layout: post
title: "Security Comparison: AngularJS vs Backbone.js vs Ember"
date: 2015-05-27
---

*Originally published on the [Software Secured blog](https://web.archive.org/web/20211129195211/https://www.softwaresecured.com/author/jbuis/). Mirrored here for posterity.*

## Comparing JavaScript Frameworks

Client side JavaScript security is becoming more and more of an issue with the shift to Single Page Applications or SPAs in modern web development. There are many different libraries and frameworks to pick from when you set out to build your own SPA.

The main players at this point come out as: Angular, Backbone and Ember.

Each framework approaches the problem of writing applications on the client side slightly differently and offer different trade-offs. The concern we want to address is the notion of security with regards to our three popularity winners.

## Quick Security Rundown

| | Angular | Backbone | Ember |
|---|---------|----------|-------|
| **Dependencies** | No Dependencies | Underscore and jQuery | jQuery and Handlebars |
| **CVEs** | 0 | 0 | 5 |
| **Retire.js** | 5 | 1 | 15 |
| **Security Policy** | No | No | Yes |
| **Security Contact** | Yes | No | Yes |
| **Security Documentation** | Yes | No | No |
| **Security Features** | Yes | No | No |

## Angular

Angular currently doesn't have any CVEs to its name and all found issues have been reported in the mailing list. Retire.js has recommendations to upgrade five different versions based on found vulnerabilities. Angular offers one page of documentation to help developers keep their code secure. There is no security policy in place. They have a contact for security issues listed on their website.

Angular handles its own templates and has no dependencies.

Angular implements their own version of JavaScript expressions which are more strict and run in a sandbox, which they call an Expression Sandbox. The sandbox is meant to maintain proper separation of application responsibilities like disallowing access to the window global object. Although not a security feature per se, being able to execute malicious code in the sandbox is one way of introducing XSS attacks into the web app.

## Backbone

Backbone doesn't have any CVEs either at the moment, and in fact no publicly reported vulnerabilities are currently available. Retire.js only has one version to upgrade from for Backbone.

Backbone itself doesn't have a security policy, nor does it offer documentation on how to write secure Backbone code. Backbone requires both jQuery and underscore. jQuery has public CVEs.

Backbone doesn't have any concept of an Expression Sandbox because it is much simpler in scope. It's up to the application developers using Backbone to take care of JavaScript expressions that get placed inside templates.

## Ember

Ember has these publicly listed CVEs:

- CVE-2013-4170
- CVE-2014-0013
- CVE-2014-0014
- CVE-2014-0046
- CVE-2015-1866

Retire.js has fifteen Ember versions to upgrade away from mostly based on the CVEs that have been publicly released and issues that have been posted to the mailing list. Ember has a security policy on their website and a security response contact.

Ember doesn't implement an Expression Sandbox either, and again it is up to the developer to use JavaScript expressions appropriately when inserting into client side templates.

## A Note on Client Side Templates

Client side templates are where injection attacks to the DOM are most prevalent. All three libraries are vulnerable to this by differing degrees. There is an excellent reference for client side templating security in [mustache-security](https://code.google.com/archive/p/mustache-security/). They provide a ranking system:

- **SEC-A**: Are template expressions executed without using eval or Function? (yes = pass)
- **SEC-B**: Is the execution scope well isolated or sand-boxed? (yes = pass)
- **SEC-C**: Can only script elements serve as template containers? (yes = pass)
- **SEC-D**: Does the framework allow, encourage or even enforce separation of code and content? (yes = pass)
- **SEC-E**: Does the framework maintainer have a security response program? (yes = pass)
- **SEC-F**: Does the Framework allow or encourage safe CSP rules to be used? (yes = pass)

| Framework | SEC-A | SEC-B | SEC-C | SEC-D | SEC-E | SEC-F |
|-----------|-------|-------|-------|-------|-------|-------|
| Angular (1.4.0) | Fail | PASS | Fail | PASS | PASS | PASS |
| Ember (latest) | Fail | PASS | PASS | Fail | PASS | TBD |
| underscore (latest) | Fail | Fail | PASS | Fail | Fail | Fail |

## Bonus Round: jQuery

jQuery is a hard dependency of both Ember and Backbone. jQuery has these public CVEs:

- CVE-2007-2379
- CVE-2010-5312
- CVE-2011-4969
- CVE-2012-6662
- CVE-2013-4383
- CVE-2015-2089

jQuery doesn't have a security response contact or a policy in place to handle vulnerabilities. The main tip for jQuery is to sanitize any user input that gets placed back into the DOM.

## Summary

It's hard to say one library is more secure than the other. They all solve similar problems in different ways with different design choices.

My recommendation comes down to being aware of the limitations and design decisions that are thrust upon you when you pick one of these three libraries.

The general recommendations are still the same for client side JS security:

- Access control, input validation and security decisions must be made on the server.
- Handle untrusted data with care — use contextual encoding and avoid building code from strings.
- Protect your services.
- Learn how to use security HTTP headers.

## References

- [NVD](https://nvd.nist.gov/)
- [retire.js](https://retirejs.github.io/retire.js/)
- [mustache-security](https://code.google.com/archive/p/mustache-security/)
- [Single Page Web App Security Cheat Sheet](https://github.com/nicola/nicola.github.io)
