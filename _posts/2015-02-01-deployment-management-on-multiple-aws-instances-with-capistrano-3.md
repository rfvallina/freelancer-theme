---
layout: post
title: "Deployment management on multiple AWS instances with Capistrano 3"
date: 2015-02-01 16:02:16 +0100
comments: true
category: blog
tags: [aws, ec2, linux, centos, git, ci]
---

When your application has to run on multiple servers or you simply has multiple applications running on different servers it's always a good idea to automate this process so you don't have to manually deploy the application on each server every time a new fix, release or whatever is applied.
<!-- more -->
In my case, in the search of optimal results I have the same version of the application running on 3 different **AWS EC2** instances on different locations (Virginia, Oregon and California). You will understand what a nightmare this is when a new version of the application has to be deployed if you do this task manually.

So I considered doing a little research on how to improve this process by running one or a few commands to make all happen automatically. And this is what I found.

- **AWS OpsWorks**: I never used it, but as far as I know it's highly customizable and useful for scaling up applications. Everything you can do with a script can be done with OpsWorks and there's no additional charge for using it. The drawback is what if you have an instance of the application running outside AWS environment. Then this is not your choice.
- **[Puppet](http://puppetlabs.com/)**: I didn't test it but I saw it consists on a web tool and you can have your deployment configuration up and running very quickly and effortlessly. The drawback is that it's too expensive for a small business. Here is the [pricing](http://puppetlabs.com/puppet/how-to-buy).
- **Jenkins**: This is a powerful deployment management tool. I used it on Java projects mainly for **CI**. I always used it by installing it on the target server and with the aim of managing the deployments on that server only. It's open source.
- **[Capistrano](http://capistranorb.com/)**: This is a Ruby tool whose purpose is to automate tasks on remote servers. Although it's perfect for our purpose, it can be good for other tasks like automate audits of any number of machines, script arbitrary workflows over SSH, automate common tasks in software teams,... It's fully customizable and it's open-source. So I decided to go with it to manage my deployments.

## Installation

First of all think about where your deployment management environment will be hosted. Personally I prefer to have the minimun possible configurations locally, so I decided to host it on my own virtual server (Linux CentOS 7). Below are the steps to install Capistrano 3 on Linux CentOS 7.

- Install Ruby (1.9+) if you haven't
``` sh
sudo yum install ruby
```

- Install Capistrano
``` sh
gem install capistrano
```

- Create project folder
``` sh
mkdir -p deployment/my-project
cd deployment/my-project
```

- Install capfile
``` sh
cap install
```

This creates the following files:
```
├── Capfile
├── config
│   ├── deploy
│   │   ├── production.rb
│   │   └── staging.rb
│   └── deploy.rb
└── lib
    └── capistrano
            └── tasks
```

## Usage

- Set global variables on **deploy.rb**

``` ruby
set :application, 'my-app-name'
# set :repo_url, 'git@example.com:me/my_repo.git'
set :repo_url, 'https://your-github-token@github.com/your-github-username/your-repo.git'

# Default branch is :master
set :branch, 'my-branch'
```

**Note 1**: I use token authentication method for Github. You can enable this on your github settings.
**Note 2**: If you don't set **:deploy_to** variable, the project is downloaded from github into **/var/www/my_app_name** with a specific structure:

```
├── current -> /var/www/my_app_name/releases/20150120114500/
├── releases
│   ├── 20150080072500
│   ├── 20150090083000
│   ├── 20150100093500
│   ├── 20150110104000
│   └── 20150120114500
├── repo
│   └── <VCS related data>
├── revisions.log
└── shared
    └── <linked_files and linked_dirs>
```

- Customize deployment tasks

There are several hook tasks e.g. **:started**, **:updated** for you to hook up custom tasks into the flow using **after()** and **before()**. In my case I created my own tasks to copy specific files from the repository download folder to the application deployment folder.

``` ruby
namespace :deploy do

	after :finished, :copy_files do
		target_dir = "/var/www/html/deploy"
		target_dir_includes = target_dir + "/includes"
		on roles(:virginia, :oregon, :california) do
	    	print "copying #{current_path} -> #{target_dir}\n"
	    	execute "cp -rf #{current_path}/* #{target_dir}"
	    end
	    on roles(:virginia) do
	    	print "copying #{target_dir_includes}/db_config_virginia.php -> #{target_dir_includes}/db_config_data.php\n"
	    	execute "cp -f #{target_dir_includes}/db_config_virginia.php #{target_dir_includes}/db_config_data.php"
	    end
	    on roles(:oregon) do
	    	print "copying #{target_dir_includes}/db_config_oregon.php -> #{target_dir_includes}/db_config_data.php\n"
	    	execute "cp -f #{target_dir_includes}/db_config_oregon.php #{target_dir_includes}/db_config_data.php"
	    end
	    on roles(:california) do
	    	print "copying #{target_dir_includes}/db_config_california.php -> #{target_dir_includes}/db_config_data.php\n"
	    	execute "cp -f #{target_dir_includes}/db_config_california.php #{target_dir_includes}/db_config_data.php"
	    end
	end
```

Note that I have three roles (virginia, oregon and california) which correspond to the locations where the application is deployed. I make use of the roles concept to have different host and tasks per each.

Below is my **production.rb** file.

``` ruby
role :virginia, %w{ec2-user@my-virginia-instance-domain.com}
role :oregon, %w{ec2-user@my-oregon-instance-domain.com}
role :california, %w{ec2-user@my-california-instance-domain.com}

server 'my-virginia-instance-domain.com',
user: 'ec2-user',
roles: %w{virginia},
ssh_options: {
     user: 'ec2-user', # overrides user setting above
     keys: %w(/path/to/virginia-file.pem),
     auth_methods: %w(publickey)
   }

server 'my-oregon-instance-domain.com',
user: 'ec2-user',
roles: %w{oregon},
ssh_options: {
     user: 'ec2-user', # overrides user setting above
     keys: %w(/path/to/oregon-file.pem),
     auth_methods: %w(publickey)
   }

server 'ec2-user@my-california-instance-domain.com',
user: 'ec2-user',
roles: %w{california},
ssh_options: {
     user: 'ec2-user', # overrides user setting above
     keys: %w(/path/to/california-file.pem),
     auth_methods: %w(publickey)
   }
```

Note that I'm using 'publickey' authentication method, so you need to tell capistrano the path to your private key (.pem file). For authentication you have to generate a public key and copy it to the remote servers. For more information about authentication on the remote servers, have a look at [capistrano reference guide](http://capistranorb.com/documentation/getting-started/authentication-and-authorisation/).