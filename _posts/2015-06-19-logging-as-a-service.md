---
layout: post
title: "Logging as a Service"
date: 2015-06-19 12:23:49 +0200
comments: true
category: blog
tags: [java, logging, saas]
---

Every software developer knows just how important it is to have a well-defined, well-functioning logging mechanism. Alas, we don’t always devote much effort to this task, thus missing on its many benefits, mainly error prevention and/or detection. If we look back in time, we readily observe that for Java the log4j library has been en vogue for a while now. Oftentimes, and until very recently, many apps were log4j-based and their log files were stored somewhere in the server. I have to admit that this could well be enough in some cases, provided the configuration is done competently and with a decent design.
<!-- more -->
However, things have been changing really rapidly, and now we have more attractive possibilities. The SaaS revolution has enabled what we call **cloud logging**, with the popularization of services such 
as [**loggr**](http://loggr.net/), [**papertrailapp**](https://papertrailapp.com/) or [**loggly**](https://www.loggly.com/). Nowadays, even servers can be easily set up, using tools like **syslogd**, while **Splunk** can be relied on to analyze our data.

<p class="text-center">
	<img class="img-responsive" src="/img/blog/Logging_as_a_Service_Architectural_Model.jpg">
</p>

Thus, delegating app logging to a third party definitely offers many advantages. I have personally tested this technique with some of my apps, and cannot but recommend it. There are few cons, and many pros. Among the most important are:

- Developers only need to decide where in their apps they want to log info. No time allotted to design or to muse on issues such as logging paths, permissions and so on. Much time can be saved this way.
- Being multi-platform systems, it pretty much doesn’t matter which programming language you are using. Your logging system remains the same, there’s no reason at all to change it, so you only need one client to connect and call the service.
- Centralized logs. Imagine an arrangement where dozens or even hundreds of apps are working to fulfil a lot of different tasks; every log is centralized, so the administrator has its hands free to monitorize all the apps. Thumbs up.

***And now, for the caveat***: cloud logging requires, before you begin, the development of a client, which will be used for making calls to the service. This client can be very simple, using methods for making HTTP calls (UDP if you are going to use sylogd) and connecting to a service’s endpoint. Here I would advise you to develop a scalable client, which can work for any logging service; thus, if you happen to be unhappy with your provider, you will be able to make the change fairly easily.

I have developed a Java **logging wrapper** and made it available in [github](https://github.com/rfvallina/log-wrapper). It’s been implemented only for loggr.net and log4j, but it could be expanded for any other system. The big upside to using this kind of wrappers is that you can switch between several logging systems simply by changing one of the parameters in the configuration. Enjoy!