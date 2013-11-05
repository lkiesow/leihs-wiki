A typical production deployment process for leihs could look like the following. This is modeled after the process that is actually in use at the Zürich University of the Arts (ZHdK).

Server landscape, the servers you're deploying to:

* rails.zhdk.ch with the following:
  * Apache
  * Phusion Passenger
  * RVM
  * Ruby 1.9.3-p448 installed through RVM

* db.zhdk.ch with the following:
  * MariaDB


Client requirements:

You need to have at least Ruby 1.9.3 and the following gems on the workstation that you will be deploying from:

* capistrano
* capistrano-ext
* rvm-capistrano


## Steps to take to deploy a new version of leihs at the ZHdK

With Capistrano installed on your workstation, enter the leihs source code directory and do:

    # cap production deploy

If you have permission to log in as user `leihs@rails.zhdk.ch`, Capistrano will magically deploy a version of leihs to rails.zhdk.ch. Where does Capistrano get those usernames and server names from? How does it know what to do? The following explains exactly what happens here and in which steps.


## Explanation of the files in config/deploy

Most of the Capistrano magic happens in the files in config/deploy. There are several of these:

### config/deploy/deploy.rb

This file is absolutely empty. This is a remnant of the capistrano-ext extensions for multi-stage deployments. You can put some toplevel configuration in here, but we don't really need that. All serious configuration is done inside the multistage config files that we'll explain next.


### config/deploy/production.rb

The production configuration file. Production here refers to the production server, as in not the staging server. Since this guide is about deploying to production, we'll look at this first.




