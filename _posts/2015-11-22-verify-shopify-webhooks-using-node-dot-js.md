---
layout: post
title: "Verify Shopify Webhooks using Node.js"
date: 2015-11-22 20:17:17 +0100
comments: true
category: blog
tags: [nodejs, ecommerce, shopify]
---

As you might know if you have experience with the [Shopify API](https://docs.shopify.com/api), Shopify provides **Webhooks**, which is a tool for retrieving and storing data from a certain event. They even allow managing Webhooks through the API. For instance, if you want your application to be notified when a new order is placed in your store, you may register a Webhook for the _create/order_ event.
<!-- more -->

But you might be wondering how to ensure that the incoming request actually comes from Shopify. The answer is yes, there's a way to guarantee that. Specially for security reasons, it's almost a must to verify webhooks. At the time of writing this there's no official documentation for [verifying webhooks](https://docs.shopify.com/api/webhooks/using-webhooks#verify-webhook) in Node.js, so I'm going to explain how to do it. It's pretty straightforward.

Webhooks can be verified by calculating a digital signature. Each Webhook request includes a **X-Shopify-Hmac-SHA256** header which is generated using the app's shared secret, along with the data sent in the request. To verify that the request came from Shopify, compute the HMAC digest according to the following algorithm and compare it to the value in the **X-Shopify-Hmac-SHA256** header. If they match, you can be sure that the Webhook was sent from Shopify and the data has not been compromised.

In this example I'm going to use the [**Express**](http://expressjs.com/) framework with the [**body-parser**](https://www.npmjs.com/package/body-parser) middleware. Basically, what the code below is doing is verifying every incoming request. If the url path matches the one for my webhook event, then it calculates the signature. If the calculated signature matches the one that comes in the header it goes ahead processing the request. Otherwise it throws an error.

```js
app.use(bodyParser.json({
    verify: function(req, res, buf, encoding) {
        if (req.url.search('shopify/webhook') >= 0) {
            var calculated_signature = crypto.createHmac('sha256', 'SHOPIFY_SHARED_SECRET')
                .update(buf)
                .digest('base64');
            if (calculated_signature != req.headers['x-shopify-hmac-sha256']) {
                throw new Error('Invalid signature. Access denied');
            }
        }
    }
}));
```