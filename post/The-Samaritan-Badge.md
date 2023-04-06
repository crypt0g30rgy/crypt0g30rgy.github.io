---
title: 'The Samaritan Bug'
metaTitle: 'The Samaritan Bug'
metaDesc: 'dev@jijo:~$whoami'
socialImage: images/samaritan.png
date: '2021-10-04'
tags:
  - first
---

# The Samaritan Badge

## The Bug

Hello Reader, Hope you are doing well. Today we will be going through my first report on a VDP.
It was another fine day when i recieved an invitation to a vdp program on hackerone that allowed hacking all their inscope assets like [*.jij0.me](/post/why)
I went on google and used the google dork `site:jij0.me filetype:aspx`. I got one subdomain that had <https://stats.home.jij0.me/aboutus.aspx>, i visted the subdomain and started polking around, i forced an application 404 error not found, then i did a view source and started poking around at the site clientside source. Thats when i saw the url `https://stats.home.jij0.me/TL/_vti_bin/spsdisco.aspx` referenced at a certain line, `The _vti_bin is the built-in SharePoint Web services directory,` i remembered having learned about an informational disclosure that could result in this folder being accessible without authentication.

```bash
Domain = *.jij0.me
subdomain = stat.home.jij0.me
Vulnerable endpoint = <https://stat.home.jij0.me/TL/_vti_bin/spsdisco.aspx>
```

I also found exposed audit logs of the same subdomain using the google dork `site:jij0.me filetype:xlsx`

`https://stat.home.jij0.me/AuditLogs/Audit_Log_2013-12-29T180000.xlsx`

## Reproduction

The issue can be reproduced by;
Visiting the following urls <https://stat.home.jij0.me/TL/_vti_bin/spsdisco.aspx> && <https://stat.home.jij0.me/iMASAuditLogs/Audit_Log_2013-12-29T180000.xlsx>

I made a report and sent it to the program and it got accepted as a medium.

I got my samaritan h1 badge.

![basic](/images/poc/triage.png)

## Further Exploitation

There is a exploitation research paper on the same services from the blackhat and they also developed a tool called sparty to further exploit it.
You can automate the exploitation via using the smarty tool from github.

```bash
python Sparty-2.0 -u https://example.com -enum -exploit 
```

[Smarty](https://github.com/MayankPandey01/Sparty-2.0)

## Impact

An attacker can leverage this information to gain foothold in the webapp and craft attacks accordingly.
You can see all the webservice endpoints which contain some sensitive information.

## Mitigation

Place access control list to the dll’s of the sharepoint.
Forbid all the folder /_vti_bin/_vti_adm/admin.dll via a WAF/Server-403 or remove it completely

## Resources

[https://hackerone.com/reports/807915](https://hackerone.com/reports/807915)
[https://hackerone.com/reports/300539](https://hackerone.com/reports/300539)

Thank You For reading my wite up. Hope you enjoyed it or learned something from it.
GoodLuck in Your Bug Hunting.

## Contacts

### @[github](https://github.com/crypt0g30rgy)  @[twitter](https://twitter.com/crypt0g30rgy) @[LinkedIn](https://www.linkedin.com/in/george-maina-waithaka-95a465214/)   @[Intigriti](https://app.intigriti.com/profile/g30rgyth3d4rk) @[hackerone_old](https://hackerone.com/crypt0p3n3tr4t0r?type=user)
