---
title: 'The Bug That Kept On Giving :: PaymentBypass :: Response Manipulation'
metaTitle: 'payment bypass'
metaDesc: 'dev@jijo:~$paid'
socialImage: images/paymentbypass1.png
date: '2022-12-16'
tags:
  - payment bypass two
---

# The Bug That Kept On Giving :: PaymentBypass :: Response Manipulation

## How we got there

A story of the third bug i found after the one for [QR CODE](/post/PaymentBypassOne) And the Next [Exposed Return URL](/post/PaymentBypassThree)

It was the same program jij0 [(why)](/post/why). jij0 had a website offering an option to buy e-giftcards. <https://jij0.be/jij0-gift> that would redirect you to <https://gifts.jij0.be>

This payment bypass is brought by the reliance of JavaSript to validate transactions.

```Javascript
function makePayment(data) {
            data['paymentMethod'] = $('.checkout__payment').html();
            $.ajax({
                method: "POST",
                url: "/v1/Payment",
                data: {paymentData: data},
                dataType: "JSON",
                success: function(response) {
                },
                error: function () {
                    alert('Ooops...');
                }
            }).then(action => {
                if (action == "Authorised") {
                    dropin.setStatus("success")
                    $.ajax({
                        method: "POST",
                        url: "/v1/HandleResponse",
                        data: {responseData: action},
                        dataType: "JSON",
                        success: function(response) {
                            window.location = response;
                        },
                        error: function (response) {
                            window.location = response;
                        }
                    });
                } else if (action == "Refused" || action == "Error") {
                    dropin.setStatus("error");
                    $.ajax({
                        method: "POST",
                        url: "/v1/HandleResponse",
                        data: {responseData: action},
                        dataType: "JSON",
                        success: function(response) {
                            window.location = response;
                        },
                        error: function (response) {
                            window.location = response;
                        }
                    });
```


if you read and analyze the above code you would find the vulnerability easy as i eventually did; 

> In the above snippet the developer assummed that server responses should be trusted, because a user had no control over what response the server would return but using a tool like burpsuite we can manipulate even server responses hence we would be able to control `action == & .setStatus` 

The one is handled by gifts.jij0.be following the below procedure;

item added to cart == user info filled and goes to checkout == user select pay with card == user enters card == transaction performed == gifts.jij0.be checks server responses {The issue lies here} == User either get redirected to cart if transaction failed or confirmation page if transaction is successful

Successful Transaction

`GET /v1/cart == POST /v1/Payment == [Response: 200 OK Data:"Authorised"] == POST /v1/HandleResponse == GET /confirmation`

Unsuccessful Transaction

`GET /nl/cart == POST /v1/Payment == [Response: 200 OK Data:"Refused"] == POST /nl/checkout/HandleResponse == GET /cart`

## Reproduction Steps:

1. connect burp with your prefered browser
2. go to https://gifts.jij0.be/ and add an item to your cart
3. proceed to https://gifts.jij0.be/cart and select checkout
4. follow the prompts to fill in all the required details and proceed to payment page.
5. select pay with card and add the test card "1337 1337 1337 1337" expiry data "03/30"
6. checkout first for camparisons sake  and observe the failed checkout that redirected you to https://gifts.jij0.be/cart 
7. go to payment again and do step 5 again but don't send request yet
8. Head over to burp and turn intercept on. send request now and make sure [POST /v1/Payment] request is captured by burp
9. right click and select intercept response to this request and forward request
10. observe the 200 OK response with the data "Refused" in the body.
11. Change "Refused" to "Authorised" and forward request, turn off intercept  to 
12. You should now recieve confirmation in mail

I made a report and sent it to the program and after a few days it got accepted as a high severity and Bounty €500 awarded.

![basic](/images/poc/accepted1.png)

## Contacts

### @[github](https://github.com/crypt0g30rgy)  @[twitter](https://twitter.com/crypt0g30rgy) @[LinkedIn](https://www.linkedin.com/in/george-maina-waithaka-95a465214/)   @[Intigriti](https://app.intigriti.com/profile/g30rgyth3d4rk) @[hackerone_old](https://hackerone.com/crypt0p3n3tr4t0r?type=user)
