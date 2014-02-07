# leihs administration and installation guide
Ramon Cahenzli <ramon.cahenzli@zhdk.ch>

## Introduction

leihs is web-based inventory handling and resource booking system. It allows users to view available equipment and place reservations through the frontend. Inventory managers and sysadmins use the backend to handle incoming reservations and manage items in the inventory.

This guide shows you how to install a leihs server. The guide is written from the perspective of a system administrator or developer. If you are interested in running leihs in your own organization but aren't a sysadmin, talk to your IT department. leihs is not intended to be installed on a client, so any software installation on your own machine isn't necessary. All you need is a web browser, the rest can be taken care of by your IT department.

Consulting and installation services are also available from independent companies supporting Free Software all around the world. Ask around for a company or individual who knows Ruby on Rails applications, you will surely find someone who can help you install leihs.

If you are such a person yourself and would like to have your services listed on the leihs project website and in this document, please write an e-mail to ramon.cahenzli@zhdk.ch.


## Installation of the base system on Debian GNU/Linux

These instructions were tested on a minimal install of Debian GNU/Linux 7.0 (wheezy). They might also work on Ubuntu. You might have to substitute `sudo su` for `su` because Ubuntu does not configure a root password, thus `su` would not work.

1. Install some build essentials, Ruby, Bundler, irb, libxslt-dev, MySQL client libraries, libxml2-dev etc.:

        # apt-get install  build-essential make git libssl-dev libxslt-dev libmysqlclient-dev libxml2-dev curl

2. Install [RVM](http://rvm.io/) and Bundler. When you're done, install Ruby:

        # rvm install 1.9.3-p448
        # gem install bundler

3. Install ImageMagick:

        # apt-get install imagemagick libmagickwand-dev


## Installation of the base system on RedHat Enterprise Linux, Fedora or CentOS

Please note that **this distribution is not officially supported**, but you are welcome to try these instructions. They were verified on CentOS 6.3.

1. Install libxslt, MySQL client libraries, libxml2, gcc and some required dependencies:

        # yum install curl gcc gcc-c++ libreadline-devel openssl-devel git libxslt-devel libxml2-devel libxml2 mysql-devel

2. Install [RVM](http://rvm.io/) and Bundler, since CentOS doesn't have an up to date Ruby version as a package. When you're done, install Ruby:

        # rvm install 1.9.3-p448
        # gem install bundler

3. Install ImageMagick:

        # yum install ImageMagick ImageMagick-devel


## Installing the platform-independent components

These steps apply for both Debian-based and RPM-based distributions.

1. Download the latest version of leihs from our [GitHub page](http://github.com/zhdk/leihs/releases). Unpack it to a convenient directory. We use the home directory of the 'leihs' user (/home/leihs) to install leihs in. Of course you can use any directory.

        # su - leihs
        $ wget https://github.com/zhdk/leihs/archive/3.0.4.tar.gz
        $ tar xvfz 3.0.4.tar.gz

2. Install the RubyGems that leihs needs. Bundler can do this automatically: 

        # su - leihs
        $ cd leihs-3.0.4
        $ rvm use 1.9.3-p448
        $ bundle install --deployment --without cucumber development

3. Configure database access for this installation of leihs. Copy the file config/database_local.yml to config/database.yml and set things up according to your needs. You will need a MySQL database for leihs (MariaDB will also work, but this guide does not cover installing MariaDB). Here is an example of a production database configuration:

        production:
           adapter: mysql2
           database: leihs_production
           encoding: utf8
           username: root
           password:
           host: localhost
           port: 3306

    You won't need more than a production section, since your leihs instance will be running in the production environment. Rails supports other environments, typically 'development' and 'test', but you won't need those unless you are developing leihs or running the test suite.

4. Create and migrate the database:

        # su - leihs
        $ cd leihs-3.0.4
        $ RAILS_ENV=production bundle exec rake db:create db:migrate db:seed

    If you have created the database by hand earlier, or if the user you're using in config/database.yml does not have privileges to create a database, skip db `db:create` part.

5. Create the temporary directories that are necessary for e.g. image uploads, temporary files etc. Make sure to create these directories so that the leihs user has write permission to them.

        $ cd /home/leihs/leihs-3.0.4
        $ mkdir -p public/images/attachments tmp/sessions tmp/cache


6. Precompile the assets (images, javascripts, etc.):

        $ cd /home/leihs/leihs-3.0.4
        $ RAILS_ENV=production bundle exec rake assets:precompile

7. Enable serving static assets by changing the setting in `config/environments/production.rb`:

        config.serve_static_assets = true

    This is only for testing! You will disable this again once you start setting up a real production environment using Phusion Passenger. See 'Setting up a production environment' below for some hints about this.

8. Start the leihs server:

        $ cd /home/leihs/leihs-3.0.4
        $ RAILS_ENV=production bundle exec rails s

    Now you should see your local leihs server at http://localhost:3000. You can log in with username "super_user_1" and password "pass".

    This gives you a test setup using the pure Ruby WebRick web server. For production setups, we recommend mod_passenger. See the "Installing a production environment" section of this guide for more information.

    Please change the super_user_1 password immediately after logging in the first time. Otherwise other people will also be able to log in using the well known default password.

9. Set up a system cronjob that sends nightly e-mail reminders and, more importantly, updates all the models' availability counts. There are many ways to schedule repeating tasks on GNU/Linux, but here's a line in crontab-format that you can add to your leihs user's crontab using e.g. `crontab -e`:

        1 00    * * *   /home/leihs/cron.sh

    Then use something like the following shell script to actually execute what's necessary. We use a shell script wrapper like this because getting RVM and crontab to cooperate is not so easy otherwise.

        #!/bin/bash

        source /usr/local/rvm/environments/ruby-1.9.3-p448
        rvm use 1.9.3-p448
        cd /home/leihs/leihs-3.0.4
        RAILS_ENV=production bundle exec rake leihs:cron


    The important bit here is to run the "leihs:cron" rake task. How you do this exactly is irrelevant.

Once you have tested that everything works correctly, make sure you read through the section below on setting up a real production environment.

## Users, logins and levels 

### Default admin username/password 

After installation, a default user is created for the Database Authentication module. Username: super_user_1. Password: pass.

Please change the super_user_1 password immediately after logging in the first time. Otherwise other people will also be able to log in using the well known default password.


### Hooking up to LDAP for logins 

Currently, leihs contains only a very rudimentary LDAP login adapter. It was developed by the Zurich University of the Arts specifically for the Hochschule der Künste Bern, Switzerland, and some parts of it are hardcoded to fit the environment there. However, it's not impossible to get leihs to authenticate against your own LDAP server if you are willing to modify two files in leihs.

#### Modifying config/LDAP.yml

Open config/LDAP.yml and adapt the configuration to your own LDAP server:

        production:
         host: your.ldap.server
         port: 636
         encryption: simple_tls
         base: dc=yourcompany,dc=com
         log_file: log/ldap_server.log
         log_level: warn
         search_field: uid
         bind_dn: *****
         bind_pwd: ******

You may have to adapt the `search_field` option to point at the LDAP attribute that contains your usernames. The `search_field` dictates what users will have to write in the "Login" field on login.

You will also have to go to the settings dialog in leihs and add the absolute path to your LDAP.yml there in the `ldap_config field.

#### Modifying app/controllers/authenticator/ldap_authentication_controller.rb

This controller handles the login itself. You will have to modify a few strings to point at the right attributes for your own LDAP server.

On line 33:

        email = users.first.mail if users.first.mail
        email ||= "#{user}@hkb.bfh.ch"

If your LDAP server does not store the e-mail address in the field "mail", change `users.first.mail` to `users.first.your_email_field_name`. You can also change `"#{user}@hkb.bfh.ch"` to `"#{user}@your.domain.com"` so that such an e-mail is automatically constructed out of the user's login plus your domain name. This is useful in case you don't store any e-mail addresses in your LDAP server at all.

On line 52:

        u.firstname = users.first["givenname"].to_s
        u.lastname = users.first["sn"].to_s
        u.phone = users.first["telephonenumber"].to_s unless users.first["telephonenumber"].blank?

You can add any Ruby code here that extracts this user information from your LDAP. Currently things are set up to point at the default fields from the InetOrgPerson class (givenname, sn, telephonenumber, etc.). This should be okay for most LDAP servers, but feel free to change the strings in e.g. `users.first["givenname"].to_s` to your own field names. For a field called "firstname", that would change to: `users.first["firstname"].to_s`.

#### Enabling LDAP authentication in the system

Finally, you need to tell leihs that you want to use LDAP, not local database authentication. Start a Rails console inside your leihs directory:

    $ RAILS_ENV=production bundle exec rails c

Then enable LDAP authentication and switch off database authentication:

        >> ldap = AuthenticationSystem.find_by_class_name("LDAPAuthentication")
        >> ldap.is_default = true
        >> ldap.is_active = true
        >> ldap.save
        
        >> db = AuthenticationSystem.find_by_class_name("DatabaseAuthentication")
        >> db.is_default = false
        >> db.is_active = false
        >> db.save

Now restart your Rails application and next time you try to access it, you should be forwarded to /authenticator/ldap/login instead of /authenticator/db/login, and then you can log in via LDAP.

Do you want to be able to configure all these settings directly in LDAP.yml so it's easier to hook up to your LDAP server? Feel free to improve our LDAP connector and send us a pull request on GitHub. Alternatively, you could also pay some Rails developers (even us!) to develop this feature for you.

Warning: Please make sure that your Rails application server has SSL enabled before you put this configuration into production. You don't want to send passwords unencrypted over the web.


### User levels and roles 

leihs decides what it lets users do based on their role and level. Each role or level is specific to an inventory pool, the only exception is the admin role, which covers the whole system.

There are three available roles: customer, manager and admin. 

#### Customer

Customers may only use the frontend and submit orders.

#### Manager level 1

Think of this manager as the "lending and borrowing manager".

* May only use the "Booking" section of the backend and parts of the "Administration" section
* May acknowledge orders for their inventory pool
* May hand over orders and create contracts
* May take back orders

#### Manager level 2

Think of this manager as a "junior manager".

* Everything that a manager at level 1 can do, plus:
* May assign levels and permissions (within their own inventory pool) up to and including level 2
* May create new models
* May create new items that are not inventory relevant, and the manager may not change this setting
 * These items have "Responsible department" set to their own inventory pool and the manager may not change this setting

#### Manager level 3

Think of this manager as a "senior manager".

* Everything that levels 1 and 2 can do, plus:
* Sees the "Inventory" section of leihs
* Has no restrictions on editing items
 * May create things that are inventory relevant
 * May assign items to any inventory pool as "Responsible department"
 * Using these functions, managers of level 3 can purchase items using their own inventory pool, but assign responsibility for the items to other inventory pools, so those other pools can manage their borrowing and lending independently
* May manage categories

#### Admin

* May manage users and assign permissions
* May manage inventory pools
* May manage groups

## Performing upgrades

### Upgrading from leihs 2.9.14 to leihs 3.0.0

This is a major upgrade that removes many dependencies from your leihs installation, but adds some others.

Since the guide is pretty long, we have covered this on a separate page: [[Upgrading-2.9.14-to-3.0.0]].

### Upgrading from older versions to leihs 2.9.14

This is described in the guide [[Upgrading-older-versions]].


## Installing a production environment 

Please note that this quick-start guide does not cover running a Ruby on Rails application for production. Please look into [Phusion Passenger](http://www.modrails.com/) for how to set up a production environment.

Without a real production environment, leihs can handle upwards of 1000 users (not concurrently, of course!). If you need better performance or want an easier to handle setup, you must set up a proper production environment.


## Free Software Statement 

The Zurich University of the Arts supports Free Software http://www.gnu.org/philosophy/free-sw.html[as defined by the Free Software Foundation]. That's why leihs is Free Software licensed under the GNU GPL version 3.0. 

One of the advantages this freedom brings with it is that it enables anyone in the world to provide local support services for leihs at the same quality level as the Zurich University of the Arts can provide itself.

If you would like to take part in the development of leihs, please see [[Contributing]].

## References 

Setting up production environments with Rails:

* http://www.modrails.com/
* http://blog.codahale.com/2006/06/19/time-for-a-grown-up-server-rails-mongrel-apache-capistrano-and-you/

## Appendices

### Appendix A: ZHdK configuration file for mod_passenger 

This configuration is recommended for anyone who is already running an Apache or nginx web server.

[Download Phusion Passenger](http://www.modrails.com) and install it according to the instructions there.

Afterwards, a simple VirtualHost entry will be enough for Apache to pick up your leihs/Rails app:


       <VirtualHost *:80>
          ServerName your.leihs.example.com
          DocumentRoot /home/leihs/public/

          <Directory /home/leihs/public>
             AllowOverride all
             Options -MultiViews
          </Directory>
   
       </VirtualHost>
