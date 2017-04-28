---
layout: post
title: "Tips to Customize Shopify Themes"
date: 2016-06-05 10:37:21 +0200
comments: true
category: blog
tags: [Shopify, eCommerce]
---

Whether you are changing the look of your theme or you are adding a new feature not supported by your current theme you have to make changes on your theme files. Shopify gives some recommendations to achieve this in [this post](https://help.shopify.com/themes/customization).
<!-- more -->

<p class="text-center">
 <img class="img-responsive" src="/img/blog/shopify.png"/>
</p>

## Before Start

You have to make a copy of the theme you want to customize (usually the online one). You will keep making changes to the copy and previewing them till you are comfortable with the results.

<p class="text-center">
 <img class="img-responsive" src="/img/blog/shopify_duplicate.png"/>
</p>

You have to be aware that changing the theme of the site can lead to problems on your site. Shopify has a support policy and [provides an explanation](https://help.shopify.com/themes/customization#getting-help) about the **supported** and **unsupported** customizations. If you are about to do unsupported customizations you should ask a Shopify expert first. 

## How to Edit

Shopify does not provide FTP service to transfer theme files. They provide access to the store files through the admin portal and the database via the API though. But their systems are not accessible via ftp for security and data integrity.

If you want to edit the theme through the Admin portal, you have several options:

1. Online editor

<p class="text-center">
 <img class="img-responsive" src="/img/blog/shopify_customize_theme.png"/>
</p>

2. Edit HTML/CSS/JS files directly

 With this method you would copy and paste the changes directly into the theme files.

<p class="text-center">
 <img class="img-responsive" src="/img/blog/shopify_edit_html_css.png"/>
</p>

3. External tool to sync theme files
 
 Here you have a powerful tool to sync files in the way Dropbox does. https://github.com/Shopify/shopify_theme

## Publish

Once you are satisfied with the results you'd want to publish the changes. You have to press **Publish theme** button.
> Test the changes thoroughly before making them public. Although Shopify keeps history of the files and you could go back to an older version it's important you test your site in depth.

## Use Git

One of my recommendations is to use a _scm_ (Source Control Management) system like **Git**. Consider using **Bitbucket**. It's free.

Before start you would make a copy of your theme on your local machine and setup a remote git repository.
As you make changes on the files you would be committing them to the remote repository. If, at some point in time you face any issue, at least you could review all changes made in the past and restore a specific version of a file.

There are interesting tools which allow you to make the whole process automatic, including deployments into Shopify. [Read this post for more information](https://www.shopify.com/partners/blog/19752835-using-git-to-simplify-shopify-theme-deployment)


