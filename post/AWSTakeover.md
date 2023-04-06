---
title: 'Owning half of a government assets through AWS'
metaTitle: 'Expos'
metaDesc: 'dev@jijo:~$whoami'
socialImage: images/aws.png
date: '2022-12-20'
tags:
  - first exceptional bug
---

# Owning half of a government assets through AWS

## How we got there

So it was on one evening, i was looking at targets at intigriti that had wide scope, `(and offered some compensation to hackers for the time invested)`.

I landed on a good government based bug bounty, `that claimed to offer a bonus to hackers`. Lets call the target jij0 [why](/post/why) it had a scope of *.jij0.me

I went a head to my Parrot Sec VM and ran [findomain](https://github.com/Findomain/Findomain) tool for gathering subdomains. I got 700+ subdomains, ran [httprobe by tomnomnom](https://github.com/tomnomnom/httprobe). It narrowed down the alive subdomains to several hundreds. I ran a simple script that i created with the help of chatgpt that sorts urls based on response types i.e 2XX, 3XX, 4XX etc and saves the to a folder called codes with corresponding url in the corresponding  file i.e 200 OK will be save to `codes/2XX.txt`

Find the tool on my github [https://github.com/crypt0g30rgy/Urls-Sorter](https://github.com/crypt0g30rgy/Urls-Sorter)

I started visiting all the 302 urls in my browser, because more than 75%  of the target redirected to an sso provider i.e visiting admin.jij0.me would redirect me to sso.jij0.me.

I observed that most of the domains were made using JavaScript Frameworks under Nodejs. As i showed earlier in my previous post ([AuthBypass](/post/AuthBypass)) if devs are not careful these frameworks can expose the web app to attacks like [PHP EAR Vulnerability](https://owasp.org/www-community/attacks/Execution_After_Redirect_(EAR)). So if a developer assumes that they are protected by the SSO and adds secrets to the Web app public config because no one will see the web app under SSo `(Oh!! How Wrong they can be)`

As i was visiting the domains i would do `view-source:{domain}` and record my config findings in a config.txt file. I found a lot of api keys for their internal apis but they all seemed to be secured, so i decided to get every possible config variable first till i had visited all domains.

In one domain lets call it `control.jij0.me` i visted redirected me as usual to the SSO, so i did `view-source:https://control.jij0.me` and i searched for api and what was returned was a goldmine `(i had no idea then)`. It returned values like below;
  
```bash
--snip--
accessKeyId:"AKIAGasfjkJFJFJKFDW"
secretAccessKey:"64cpqZwBdDTKDgkjagskgasldgh+asjfjasgdfs"
region:"eu-west-1"
awss3bucket:"s3b-dumdumbucket"
--snip--
```

I put this in the config.txt and didn't think much about it, as i had previously found keys that were restricted so i thought this too would be. I continued all my recon on configs since no authentication could be tested as we couldn't register on the SSO. After i was done, i was tired and decide to sleep and return to check the configs the following morning.

The next day i woke up, grabbed a cup of coco and went on to my work desk, after my pc powered on, i openned the config.txt file and decide to check the aws keys first. I had [enumerate-iam](https://github.com/andresriancho/enumerate-iam) installed so all i needed was to run it against the keys. I did;

`python3 enumerate-iam.py --access-key AKIA... --secret-key 64cq...`

And oh my, the tool light-up like a christmass tree. almost every call returned `status: worked!`

I had access to almost half their aws cloud assets, including hundreds of s3 buckets, several ec2 instances etc, full of citizens data.

## Reproduction Steps

1. Visit <https://control.jij0.me/> .
2. As observed, you get redirected to <https://sso.jij0.me/as/authorization.oauth2?client_id=control&response_type=token&scope=openid&redirect_uri=https://control.jij0.me/home>.
3. We have to stop the redirect so we can explore the source.
4. do view-source:<https://control.jij0.me/> does the trick.
5. Now time to read the main JS file <https://control.jij0.me/main.js>
6. do ctrl + f and search aws, you get taken to this line

```Javascript
const i={production:!0,base_url:"https://control.jij0.me/home",redirect_url:"https://control.jij0.me/as/authorization.oauth2?client_id=cdsinternal&response_type=token&scope=openid&redirect_uri=https://control.jij0.me/home",logoutURL:"https://control.jij0.me/idp/startSLO.ping?id_token={_token}&TargetResource=https://control.jij0.me/",nodeEndPoint:"https://api-control.jij0.me/cds",reportAPI:"https://api-control.jij0.me/cds",getSeedListApi:"https://8w839qmud.execute-api.eu-west-1.amazonaws.com/contact/contact/",accessKeyId:"AKIAGasfjkJFJFJKFDW",secretAccessKey:"64cpqZwBdDTKDgkjagskgasldgh+asjfjasgdfs",s3Bucket:"s3b-dumdumbucket",private_history_api:"http://control-gasfghashasc9070808.elb.eu-west-1.amazonaws.com:9003/dv/selectiontool/history",queryResultGraphQLTable:"allCdsSelectiontoolQueryResults",autoRefreshQuery:!1}}
```

## Playing with the aws keys

1. Now create an aws profile

`aws --profile jij0 configure`

2. Get user

 `aws --profile jij0 sts get-caller-identity`

```bash
{
    "UserId": "AIDA4HAH654QHJAFSJFA",
    "Account": "000000000",
    "Arn": "arn:aws:iam::841140266037:user/control-01"
}

```

3. list s3

 `aws --profile jij0 s3 ls`

4. list ec2

`aws --profile jij0 ec2 describe-instances`

### Attacker could

`>-Read and control Your aws Resources`

`>-execute code in any ec2`

`>-launch aws resources`

## Report

After i found this bug i was super excited and shaking a little because i was in a goverment server viewing so much data.

`
I had access to internal communications that exposed sensitive information.
`

So i wrote up a detailed report fast and sent it to the program at intigriti. The report was triaged within 45 mins.

After waiting for five days it got accepted, no bonus/bounty for this or even a hi and thanks for securing us, i guess they we not so happy to get pawned.

`As of today the domain is not responding and the keys were added 2FA.`

> On Jan 31, i got awarded a bonus. Turns out they found this blog, how, i still have no idea, i guess if the government wants to find something they will, hehe.

![basic](/images/poc/awsbug.png)

## Contacts

### @[github](https://github.com/crypt0g30rgy)		@[twitter](https://twitter.com/crypt0g30rgy)	@[LinkedIn](https://www.linkedin.com/in/george-maina-waithaka-95a465214/)   @[Intigriti](https://app.intigriti.com/profile/g30rgyth3d4rk) @[hackerone_old](https://hackerone.com/crypt0p3n3tr4t0r?type=user)
