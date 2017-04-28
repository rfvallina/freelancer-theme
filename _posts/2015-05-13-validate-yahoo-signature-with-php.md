---
layout: post
title: "Validate Yahoo signature with PHP"
date: 2015-05-14 12:53:20 +0200
comments: true
category: blog
tags: [PHP, Ruby, Yahoo]
---

Yahoo allows applications to be integrated on Yahoo sites for merchants or simply Yahoo site owners. When you develop an application for Yahoo, first thing you have to do is to be able to validate the Yahoo signature to ensure all requests come from a Yahoo site.
<!-- more -->
The [documentation](https://github.com/lexity/platform/wiki/API-Documentation) about developing an application for Yahoo is not really good but at least the section where it explains how to validate the signature comes with an example written in Ruby.

Below is the **Ruby** code to validate the Yahoo signature.

```ruby
require "digest/md5"
require "cgi"

def signature(app_secret, parameters)
  sorted_params = parameters.collect{|k,v| "#{k}=#{v}"}.sort.join
  Digest::MD5.hexdigest(app_secret + sorted_params)
end

def valid_signature?(query_params)
  provided_signature = query_params.delete("signature")
  query_params.each{|k,v| query_params[CGI.unescape(k)] = CGI.unescape(v)}
  provided_signature == signature("sample_app_secret", query_params)
end

# Assume that the query parameters arrive in a hash (while still url-encoded), like:
query_params = {
 "token" => "735c433875c919a4a7bc2b40e695b809",
 "app_token" => "sample_app_key",
 "store_url" => "http%3A%2F%2Fexample-store.com",
 "pid" => "a123b456",
 "timestamp" => "1346884490",
 "signature" => "3f1e8aba4426ff6f5d5bf579a4204e60"
}
```
And here is the **PHP** code.

```php
function validSignature($queryParams){
	$providedSignature = $queryParams['signature'];
	unset($queryParams['signature']);
	foreach($queryParams as $key=>$value){
		$queryParams[urldecode($key)] = urldecode($value);
	}
	return signature("sample_app_secret", $queryParams) === $providedSignature;
}

function signature($appSecret, $queryParams){
	$params = array();
	foreach($queryParams as $key=>$value){
		$params[] = "$key=$value";
	}
	sort($params);
	$sortedParams = implode($params);
	return md5($appSecret . $sortedParams);
}

//Assume that the query parameters arrive in a hash (while still url-encoded), like:
$queryParams = {
 "token" => "735c433875c919a4a7bc2b40e695b809",
 "app_token" => "sample_app_key",
 "store_url" => "http%3A%2F%2Fexample-store.com",
 "pid" => "a123b456",
 "timestamp" => "1346884490",
 "signature" => "3f1e8aba4426ff6f5d5bf579a4204e60"
}
```
Decide yourself which is more elegant, though I have my own opinion.

Here is a funny video which briefly explains some differences between PHP and Ruby.

<iframe width="560" height="315" src="https://www.youtube.com/embed/g9CmE_miCaE" frameborder="0" allowfullscreen></iframe>
