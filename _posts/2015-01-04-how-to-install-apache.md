---
layout: post
title: "How to install Apache, PHP 5.5 and Memcached on AWS EC2"
date: 2015-01-04
comments: true
category: blog
tags: [aws, ec2, linux, memcache]
---

AWS EC2 looks like a good choice when you need a clean instance ready for you to manage your own server as per your needs.
Recently, I launched an EC2 instance from a Linux AMI (**Amazon Linux AMI 2014.09.1**) to create my own web server with **Apache** and support for **PHP 5.5**. I know Amazon has the [EB (Elastic Beanstalk)](http://aws.amazon.com/elasticbeanstalk/?nc2=h_ls) service which comes with the necessary stuff already installed to run your Java, Node, PHP,... applications so that you only need to upload your code to the server and EB will do the rest. 
<!-- more -->
I have experience dealing with AWS EB in the past and the reason this time I chose EC2 instead of EB is that with EC2 you have total freedom to set up your server. For instance, I found difficulties setting up cron jobs on EB in the past, but with EC2 this is a simple administration task when you connect to the server through ssh.

**What were the reasons that led me to choose AWS instead of a different hosting provider?**


This is a typical question when you are looking for the best and cheapest hosting provider for your applications. My reasons here were two: scalability and location.

- **Scalability**: With AWS is easy to scale up your applications in a timely manner and with very low effort.
- **Location**: With AWS you can choose where to run your applications among different locations available all over the world or even replicate your application in different locations at the same time. The latter is what I wanted.

Below are the steps to configure your EC2 instance based on a Linux AMI to run **Apache Web Server**, **PHP 5.5**, **Memcached server** and **memcache extension**. The sources I gathered information from were basically the [AWS guid to install a LAMP server](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/install-LAMP.html) and other forums around.

- Connect to your instance using ssh. Make sure you have previously downloaded and securely stored your private key and the ssh service enabled in your security group

``` sh
ssh -i "/path/to/private_key.pem" ec2-user@your-public-dns-name
```
- Update instance packages

``` sh
sudo yum update -y
```

- Install **Apache 2.4**, **PHP 5.5** and **PDO** support for MySQL

``` sh
sudo yum install php55 php55-pdo php55-mysqlnd
```

- Start Apache and add httpd service to automatically start after a system boot.

``` sh
sudo service httpd start
sudo chkconfig httpd on
```

- Connect to your Apache server. Make sure you have previously added **port 80** to your security group in the EC2 Console. Open a browser and type the following.

http://your-public-dns-name.com/

You will see the web server welcome page.

- Create www group, add ec2-user to the group and exit the terminal for the changes to take effect

``` sh
sudo groupadd www
sudo usermod -a -G www ec2-user
exit
```

- Give www ownership and add permission to the document root (/var/www)

``` sh
sudo chown -R root:www /var/www
sudo chmod 2775 /var/www
find /var/www -type d -exec sudo chmod 2775 {} +
find /var/www -type f -exec sudo chmod 0664 {} +
``` 

- Test PHP with the **phpinfo** page

``` sh
echo "<?php phpinfo(); ?>" > /var/www/html/phpinfo.php
```

See the page by typing the url in a browser:

http://your-public-dns-name.com/phpinfo.php

- MySQL

I didn't install MySQL on the EC2 instance itself, but I used the [AWS RDS service](http://aws.amazon.com/rds/?nc2=h_ls), so that all instances of my application can share only one database instance.

- Install **memcached server** and add it to the system boot. By default it will be listening on localhost and port 11211

``` sh
sudo yum install memcached
sudo service memcached start
sudo chkconfig --level 345 memcached on
``` 

- Install **memcache extension**

``` sh
sudo yum install php55-pecl-memcache
```
