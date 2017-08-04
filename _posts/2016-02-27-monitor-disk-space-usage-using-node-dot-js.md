---
layout: post
title: "Monitor disk space usage in AWS using Node.js"
date: 2016-02-27 17:21:09 +0100
comments: true
category: blog
tags: [nodejs, aws, linux]
---

Sometimes, if you are a server administrator or you simply are an application owner but you have full rights on the server, you could be interested in being informed when the disk space is reaching the maximum capacity.
<!-- more -->

In my case I had to deal with this while setting up different AWS environments. In AWS there's no built-in alarm to monitor disk usage. Depending on the case, AWS could provide scripts to create custom metrics. Such is the case with **disk usage** metrics. AWS provides a [set of scripts](http://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/mon-scripts.html) to report memory, swap, and disk space utilization metrics for a Linux instance.

### Drawback
- You have to install these scripts manually
- AWS charges you for having custom metrics

### Solution
A cheaper solution is to develop your own tool to report disk usage. I created a simple Node script which sends an email notification in case the available disk percentage falls below a certain threshold.

```js
'use strict'

var disk = require('diskusage'),
	nodemailer = require('nodemailer'),
	sendmailTransport = require('nodemailer-sendmail-transport');

var THRESHOLD = 30;

disk.check('/', function(err, info) {
	var availablePercentage = ((info.available / info.total) * 100).toFixed(2);
	if(availablePercentage <= THRESHOLD){
		//Setup email data
		var transporter = nodemailer.createTransport(sendmailTransport({
			path: '/usr/sbin/sendmail'
		}));
		var mailOptions = {
		    from: 'EC2 Admin <admin-ec2@no-reply.com>', // sender address
		    to: 'address1@domain.com, address2@domain.com', // list of receivers
		    subject: 'Disk Usage Warning!!!', // Subject line
		    text: 'Only ' + availablePercentage + '% of disk space is available' // plaintext body
		};
		//Send email
		transporter.sendMail(mailOptions, function(error, info){
		    if(error){
		        return console.log(error);
		    }
		    console.log('Message sent: ' + info.response);
		});
	}
});

```

As you see, the script requires to use three third-party node modules. **diskusage** is responsible to check the disk space, while **nodemailer** and **nodemailer-sendmail-transport** are responsible for sending the email notification. It assumes that sendmail is installed in the system to ensure the message reaches its destination.

Below is the **package.json** used to setup the script. You have to type **npm install** to have all the necessary modules installed.
And type **node app.js** to run the script.

```json
{
  "name"        : "disk-usage-alarm",
  "version"     : "0.0.1",
  "private"     : "true",
  "description" : "",
  "author"      : "rfvallina",
  "homepage"    : "http://rfvallina.com",
  "dependencies": {
    "diskusage": "^0.1.4",
    "nodemailer": "^2.0.0",
    "nodemailer-sendmail-transport": "1.0.0"
  }
}
```

### Cron setup
If you want the script to run on a regular basis, let cron daemon do it. Below are the steps to add it to crontab.

- Edit crontab 

```sh
crontab -e
```

- Add entry to crontab. Place the below text in crontab.

0 * * * * node /path/to/script/app.js



