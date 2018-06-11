---
layout: post
title: "Build your ecommerce store"
date: 2018-06-10 00:00:00 +0200
comments: true
category: blog
tags: [ecommerce, vue, prestashop, palbin, shopify, moltin, snipcart, 3dcart, opencart, bigcommerce, ebay, amazon]
published: true
---

Potential clients come to me and ask how to build an online shop. That's a good question but every case must be analyzed individually. There are plenty of choices and you have to choose the one that fits best for your needs. Typical questions that come up are: Do I have to build one from scratch? How do I promote my products? How do I keep inventory synchronized if I sell from different locations?...
<!-- Read More -->

Typically, there are 5 possible scenarios to choose from. I'm going to describe each of them so that it might help to make up your mind based on your own circumstances.

<p class="text-center">
 <img class="img-responsive" src="/img/blog/133380-unsplash.jpg"/>
 <small><i>Photo by Álvaro Serrano on Unsplash</i></small>
</p>

### Full Development

Developing a fully customized ecommerce site is very time-consuming and expensive. You have to design your database of products, implement the logic of the backoffice and the store, design a good template for the storefront, invest time in SEO and marketing, host your application in a server, maintenance,...

There are services like [**moltin**](https://snipcart.com/) or [**snipcart**](https://snipcart.com/) which turn out to be fully API-driven which makes the development much more pleasant. You have to develop the front-end and back-end of your ecommerce solution integrating with their api's. The backend can be as complex (or as simple) as you want. No need to build something super revolutionary if you only want to handle your backoffice yourself.

It's uncommon that you need to build a fully customizable ecommerce solution unless you have special business conditions.

### Partial/Combined Solution

Sometimes you need to have full freedom to develop one part of your solution. For instance if you have already your website and your custom html templates you might want to have a third-party backend solution only. 

#### Backend

There are pre-built storefront packages out there that might require small customizations. This means that you have to implement "only" the backend logic or maybe to integrate with a pre-built backend API software like **moltin** or **snipcart**.
This option, as in the previous one, requires developers to set everything up and running so it turnes out to be an expensive choice as well.

<p class="text-center">
 <img class="img-responsive" src="/img/blog/snipcart.png"/>
</p>

A good storefront could be [**Vue storefront**](https://github.com/DivanteLtd/vue-storefront) which is based on **Vue** framework. I might talk about it in future posts...

<p class="text-center">
 <img class="img-responsive" src="/img/blog/vue-storefront.jpeg"/>
</p>

#### Frontend

As opposed to the previous option, you are looking for a third-party backend solution only. An interesting choice could be [**3dcart**](https://www.3dcart.com/). You have total freedom to develop your storefront as you want. You could use your own custom templates or even use something like **Vue storefront**.

### Self Hosted Solution 

A self-hosted solution means that you have to download the software and install it on your own server. I'm going to talk about 2 of the most well-known platforms: **Prestashop** and **Opencart**.

[**Prestashop**](https://www.prestashop.com) is a CMS (Content Management System) to build ecommerce stores. Everything is ready to work out of the box (as long as everything fits by default in your business requirements). It's likely that you have to do some modifications, but at least you have a working ecommerce site from the beginning.
It has the option of a cloud solution, but if you are looking for total freedom to manage the store and add extensions, I recommend to go with the self-hosted solution.
Another advantage is that it's "free". I put quotes because there are some expenses that sometimes are unknown by most of the people.
The package itself is free and open source, but you have expenses like:

- Hosting -> You need a server to host the application. And maybe a domain name if you don't have one yet.
- Setup / Maintenance -> You need somebody with technical profile to install and configure the application initially. Also, in the long term you will need maintenance.
- Payment provider fees -> In order for your clients to be able to pay the items from your website you need to pay a credit card provider or Paypal or whatever you choose to accept clients payments.
- SEO -> You could invest 0 effor in SEO or some thousands of dollars. Whatever your prefer. But this is never ending world if you need to promote your products and be the first in search engine results.
...

I'm not saying that Prestashop is not a good choice, but you have to forget about the concept of "everything is free".

[**Opencart**](https://www.opencart.com/) is an open source ecommerce solution which has also multi-store capabilities. It means that you can have multiple selling channels. It can be quite common that you have multiple stores. You can focus one of the stores to sell only peripherals and another store to sell laptops and computers. You can choose which products to sell in both and which not. Opencart lets you do this out of the box.
It's a free software, however similarly to Prestashop you have to keep in mind extra costs like: hosting, extensions, payment providers, maintenance,... It's highly likely that you need extensions to complete your ecommerce with all the desired capabilities. Extensions can be expensive or at least, if you have to install a bunch of them it can be costly.

There are much more solutions in this category, but I just want to give an insight.

<p class="text-center">
 <img class="img-responsive" src="/img/blog/opencart.png"/>
</p>

### Cloud solution

Belonging to this category are those solutions that you don't have to care about installation or hosting. You only paid a fee to open an account and use it. Interesting platforms in this category are:

#### [**Shopify**](https://www.shopify.com/)

- Impressive variety of themes
- Lots of apps to cover any missing feature that the platform could have
- Rich Rest API with a very good documentation
- 3 Plans to choose from starting at $29
- Transaction fees ranging from 0.5% to 2%
- 24/7 Support

#### [**Bigcommerce**](https://www.bigcommerce.com/)

- Themes don't look as professional as Shopify ones.
- Less Apps available in the store than Shopify
- Rest API
- 3 Plans to choose from starting at $29
- No transaction fees
- 24/7 Support

#### [**Palbin**](https://www.palbin.com/)

- Very poor variety of themes to choose from
- No apps
- No API
- 3 Plans to choose from starting at 30€
- No transaction fees
- 24/7 Support

Shopify and Bigcommerce are conceptually very similar. Palbin is more a "familiar" solution. 
If I have to choose one, Shopify is the most professional one, but it has hidden fees (they charge you up to 2% per sale). Bigcommerce is also a good option as long as you find themes that fit well for your business and you don't need to translate your site to a non-english spoken language.

### Marketplace

A **marketplace** is a big platform where multiple stores coexist. Typical examples are **eBay** and **Amazon**. I'm very used to work with both of them from an integrator perspective (using API's). They are conceptually different. Amazon is more focused on **Products** and ebay is focused on **Listings**.
If you are looking for a specific product, it's easier to find it in Amazon, but if you are looking for a deal, go to ebay.

You can build your store within a marketplace. It has a lot of **advantages** like:

- You don't have to worry about templating
- It's a cloud solution so you don't care about hosting
- You don't have to care about SEO, since you take advantage of the marketplace visibility

On the other hand, the main **disadvantage** is the multiple fees and comissions. If you don't have a high profit margin or very high sales, it doesn't worth it.

<p class="text-center">
 <img class="img-responsive" src="/img/blog/ebay.webp"/>
 <img class="img-responsive" src="/img/blog/amazon.png"/>
</p>


