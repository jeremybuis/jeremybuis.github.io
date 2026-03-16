---
layout: post
title: "Passwords Shouldn't Be Hard, Don't Cause Your Next Breach"
date: 2018-09-10
---

*Originally published on the [Software Secured blog](https://web.archive.org/web/20211129195211/https://www.softwaresecured.com/author/jbuis/). Mirrored here for posterity.*

OWASP has identified authentication and poor session management as a critical issue as far back as 2004. To this day, it is still a major issue and looks to continue to be for the foreseeable future. The main benefactor of this are data breaches, as attackers have a growing pool of possible username/password combinations to use in brute force type attacks against user accounts.

Data breaches happen all the time — roughly **19,280 records are stolen every hour** — and they should be considered the new normal. It is now a matter of *when* your company will be hit by a breach, rather than *if* at all.

## The Problem: Insecure Password Storage

As more and more data breaches occur, we find that companies are still not implementing passwords in a secure manner. Insecure storage can be caused by a number of reasons including:

- Plain-text passwords
- Storing passwords using reversible encryption
- Using weak hashing algorithms like MD5, ROT13 or SHA1
- Lack of unique salt per user password

These, coupled with poor password policies, lead to user accounts being taken over by malicious users. A poor password policy would allow passwords to be less than 8 characters, have no symbols, numbers or special characters.

**A strong password policy:**
- Should include more than 8 upper and lowercase characters
- Should include numbers
- Should include symbols

## Two Factor Authentication

Companies have been adopting two factor authentication (2FA) as it provides another level of protection for users in the case of a user's password being leaked via a data breach. 2FA consists of a one time code transmitted to the user via SMS, email, or by a piece of hardware or software.

Problems occur when the one time tokens are transmitted over insecure channels, like email and SMS.

### SMS as a Channel

SMS messages are sent in clear text and attackers at times are able to recover a user's text message, either by intercepting the message or gaining access through a different channel.

There are a number of ways an attacker can gain access to the user's phone number:

- Someone can walk into any retail cellular store and ask an employee to move your number to a new SIM. The employee will verify the person using some form of government ID, but won't necessarily verify that they own the rights to that phone number.
- The attacker can call a phone company and ask to move your number to another carrier.
- An attacker can gain access to your email, where you forward your SMS messages to. Once they see the SMS code, they can reset your password and take over the account.

## Federated Authentication: OAuth2 and SAML

Companies are adding the ability for users to use their Google or Facebook accounts to authenticate. This helps users remember one less username/password combination. This is a move in the right direction, but comes with some problems. It's hard to implement these federated identity protocols in a secure manner.

**OAuth2** is probably the most popular protocol in use. The protocol is securely designed, but is often implemented poorly. For example, Facebook fell prone to poor implementation as an attacker was able to steal a user's authorization tokens by breaking out of the OAuth2 `redirect_uri` parameter, thereby sending the user's authorization token to a server they controlled.

Some common flaws in the protocol implementation include: allowing attackers to set any `redirect_uri`, improper error handling, and problems with the `state` parameter.

**SAML** is the popular protocol in use to authenticate and authorize users for a service. It passes messages using XML. Most attacks targeted towards SAML are XML based attacks. A SAML exploit was released earlier this year which allowed malicious users to bypass the authentication process completely. This happened as a result of XML libraries incorrectly parsing XML comments, allowing attackers to inject valid usernames into a SAML assertion.

## Recommendations

- If you choose to manage the identity of your users, use a **key stretching hashing algorithm** like PBKDF2, bcrypt or argon2, with a **unique long salt per user** and a **strong password policy**.
- Use services like [haveibeenpwned](https://haveibeenpwned.com/) to notify users when their credentials have been leaked.
- If you are able to manage your application without usernames and passwords by implementing authentication using **third party identity providers** like Facebook or Google, it's highly recommended to go this route.
- **Do not use SMS** as an out of band mechanism to transport one time tokens. Use a hardware token like a YubiKey or software based token applications like Duo, Authy or Google Authenticator.

## References

- [Duo Finds SAML Vulnerabilities Affecting Multiple Implementations](https://duo.com/blog/duo-finds-saml-vulnerabilities-affecting-multiple-implementations)
- [A Breakdown of the New SAML Authentication Bypass Vulnerability](https://developer.okta.com/blog/2018/02/27/a-breakdown-of-the-new-saml-authentication-bypass-vulnerability)
- [How Often Do Data Breaches Occur?](https://itsecuritycentral.teramind.co/2017/07/26/how-often-do-data-breaches-occur-infographic)
