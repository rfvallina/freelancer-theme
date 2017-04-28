---
layout: post
title: "How to Schedule a Daily Reboot on an AWS EC2 Instance"
date: 2016-04-10 19:54:44 +0200
comments: true
category: blog
tags: [AWS, Linux]
---

I don't think this is a usual thing in production environments, but I had the requirement to reboot the AWS EC2 instance every day because the applications were slowing down as the days go by.
<!-- more -->

<p class="text-center">
 <img class="img-responsive" src="/img/blog/aws.png"/>
</p>

The standard way AWS provides is by monitoring items like CPU utilization, memory used,... This monitorization is associated with [CloudWatch](https://aws.amazon.com/cloudwatch/?nc1=h_ls). The problem here is that you get charged if you exceed the free tier limits.

There's an easy way to setup a daily reboot with **Cron** and doing a simple AWS tunning. Below I will show you how to do it. It's pretty straightforward. Note that you need to have the [aws-cli](http://docs.aws.amazon.com/AWSEC2/latest/CommandLineReference/set-up-ec2-cli-linux.html) installed.

- Go to [IAM Console](https://console.aws.amazon.com/iam/home) 
- Create a new user. This user will perform the operations. In our case, the EC2 instance reboot.
- Attach a policy with the necessary privileges to the user. The easy way is to attach the Administrator policy, so the user will have rights to perform any operation on the instance. Another option is to create a new policy and assign actions like "rebootInstance", "startInstance", "stopInstance".
- Open a terminal and login to your EC2 instance.
- Setup your AWS credentials and default region (optional)

```sh
aws configure
```

- Tell AWS to reboot the specified instance.

```sh
aws ec2 reboot-instances --instance-ids <instance-id>
```


If you need to do this daily, it's better to tell cron to do this for you. Simply add the below entry to crontab and cron will run the command every day at 9am.

**0 09 * * * aws ec2 reboot-instances --instance-ids _<instance-id\>_**

And that's it!






