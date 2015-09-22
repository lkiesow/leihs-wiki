# leihs administration and installation guide
Ramon Cahenzli <ramon.cahenzli@zhdk.ch>

## Introduction

leihs is web-based inventory handling and resource booking system. It allows users to view available equipment and place reservations through the frontend. Inventory managers and sysadmins use the backend to handle incoming reservations and manage items in the inventory.

This guide shows you how to install a leihs server. The guide is written from the perspective of a system administrator or developer. If you are interested in running leihs in your own organization but aren't a sysadmin, talk to your IT department. leihs is not intended to be installed on a client, so any software installation on your own machine isn't necessary. All you need is a web browser, the rest can be taken care of by your IT department.

## Commercial support

Consulting and installation services are also available from independent companies supporting Free Software all around the world. Ask around for a company or individual who knows Ruby on Rails applications, you will surely find someone who can help you install leihs.

Companies known to offer commercial support for leihs can be found on the [respective wiki page](https://github.com/zhdk/leihs/wiki/Commercial-support). You can also add your company there.


## Installation of the base system on Debian GNU/Linux

These instructions were tested on a minimal install of Debian GNU/Linux 7.0 (wheezy). They might also work on Ubuntu. You might have to substitute `sudo su` for `su` because Ubuntu does not configure a root password, thus `su` would not work.

1. Install some build essentials, Ruby, Bundler, irb, libxslt-dev, MySQL client libraries, libxml2-dev etc.:

        # apt-get install build-essential make git libssl-dev libxslt-dev libmysqlclient-dev libxml2-dev curl imagemagick libmagickwand-dev

2. Install [RVM](http://rvm.io/) and Bundler. When you're done, install Ruby:

        # rvm install 2.1.5
        # gem install bundler


## Installation of the base system on RedHat Enterprise Linux, Fedora or CentOS

Please note that **this distribution is not officially supported**, but you are welcome to try these instructions. They were verified on CentOS 6.3.

1. Install libxslt, MySQL client libraries, libxml2, gcc and some required dependencies:

        # yum install curl gcc gcc-c++ libreadline-devel openssl-devel git libxslt-devel libxml2-devel libxml2 mysql-devel ImageMagick ImageMagick-devel

2. Install [RVM](http://rvm.io/) and Bundler, since CentOS doesn't have an up to date Ruby version as a package. When you're done, install Ruby:

        # rvm install 2.1.5
        # gem install bundler

## Installing the platform-independent components

These steps apply for both Debian-based and RPM-based distributions.

1. Download the latest version of leihs from our [GitHub page](http://github.com/zhdk/leihs/releases). Unpack it to a convenient directory. We use the home directory of the 'leihs' user (/home/leihs) to install leihs in. Of course you can use any directory.

        # su - leihs
        $ wget https://github.com/zhdk/leihs/archive/n.n.n.tar.gz
        $ tar xvfz n.n.n.tar.gz

2. Install the RubyGems that leihs needs. Bundler can do this automatically:

        # su - leihs
        $ cd leihs-n.n.n
        $ rvm use 2.1.5
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
        $ cd leihs-n.n.n
        $ RAILS_ENV=production bundle exec rake db:create db:migrate db:seed

    If you have created the database by hand earlier, or if the user you're using in config/database.yml does not have privileges to create a database, skip db `db:create` part.

5. Generate a new secret (for encrypting cookies) and configure it in `config/initializers/secret_token.rb`:

        $ RAILS_ENV=production bundle exec rake secret

    This gives you a secret. Then put it in secret_token.rb:

    Leihs::Application.config.secret_key_base = 'YOUR_SECRET_HERE'

6. Create the temporary directories that are necessary for e.g. image uploads, temporary files etc. Make sure to create these directories so that the leihs user has write permission to them.

        $ cd /home/leihs/leihs-n.n.n
        $ mkdir -p public/images/attachments tmp/sessions tmp/cache

Note: the pre-existing directory `./log` will need write permission for logs to be written as well. It is useful for debugging but should not be given write permission in production, as the resultant logs are quite verbose and take lots of space over time.

7. Precompile the assets (images, javascripts, etc.):

        $ cd /home/leihs/leihs-n.n.n
        $ RAILS_ENV=production bundle exec rake assets:precompile

8. Enable serving static assets by changing the setting in `config/environments/production.rb`:

        config.serve_static_assets = true

    This is only for testing! You will disable this again once you start setting up a real production environment using Phusion Passenger. See 'Setting up a production environment' below for some hints about this.

9. Start the leihs server:

        $ cd /home/leihs/leihs-n.n.n
        $ RAILS_ENV=production bundle exec rails s

    Now you should see your local leihs server at http://localhost:3000. You can log in with username "admin" and password "password".

    This gives you a test setup using the pure Ruby WebRick web server. For production setups, we recommend mod_passenger. See the "Installing a production environment" section of this guide for more information.

    Please change the admin password immediately after logging in the first time. Otherwise other people will also be able to log in using the well known default password.

10. Set up a system cronjob that sends nightly e-mail reminders and, more importantly, updates all the models' availability counts. There are many ways to schedule repeating tasks on GNU/Linux, but here's a line in crontab-format that you can add to your leihs user's crontab using e.g. `crontab -e`:

        1 00    * * *   /home/leihs/cron.sh

    Then use something like the following shell script to actually execute what's necessary. We use a shell script wrapper like this because getting RVM and crontab to cooperate is not so easy otherwise.

        #!/bin/bash

        source /usr/local/rvm/environments/ruby-2.1.5
        rvm use 2.1.5
        cd /home/leihs/leihs-n.n.n
        RAILS_ENV=production bundle exec rake leihs:cron


    The important bit here is to run the "leihs:cron" rake task. How you do this exactly is irrelevant.

Once you have tested that everything works correctly, make sure you read through the section below on setting up a real production environment.

## Users, logins and levels

### Default admin username/password

After installation, a default user is created for the Database Authentication module.
Username: admin
Password: password

Please change the admin password immediately after logging in the first time. Otherwise other people will also be able to log in using the well known default password.

If logging in does not work, you should try deleting your browser cache / cookies. I had trouble especially with the admin user in Firefox 32.x.

### Hooking up to LDAP for logins

It is possible to use LDAP for logins. The procedure is described under [[LDAP configuration]].

## Performing upgrades

Almost all Leihs upgrades work the same way:

* Put the new leihs source code somewhere.
* Copy the following things from your old leihs installation to your new installation:
 * `config/database.yml`
 * `config/initializers/secret_token.rb`
 * `config/LDAP.yml` (if it exists and is used by your installation)
 * `public/images/attachments`
* Run in the new leihs directory: `RAILS_ENV=production bundle exec rake db:migrate`
* Restart your leihs server (how you do this depends on how you're running leihs, whether standalone, through Phusion Passenger, etc.)

If you run leihs straight from git, you can of course also just switch to the newest release tag right in your source code directory. Substitute the latest released version for n.n.n.:

        $ git fetch
        $ git reset --hard n.n.n
        $ bundle install
        $ RAILS_ENV=production bundle exec rake db:migrate

But again, this only works if you've checked out leihs straight from git before. This won't work on a leihs installation unpacked from a tarball.


## Upgrade stepping stones from very old versions

If your leihs version has not been updated in quite some time, you will need to use a combination of different older versions of tools and you will have to reach specific stepping stones in leihs history so that you can upgrade to the very latest version.

The sequence that works is:
 * Anything older than 2.9.14 to 3.0.0
 * 3.0.0 to 3.5.0
 * 3.5.0 to 3.14.0
 * 3.14.0 to 3.23.0
 * 3.23.0 to the latest version

It's necessary to use a specific (and outdated) version of Rubygems to be able to install all the gems for specific (and also outdated) versions of leihs. The following table shows which Rubygems version is needed at which point:

| from leihs |to leihs  | Rubygems version |
|---|---|---|
|< 3.5.0  | 3.5.0 | 2.0.12  |
|< 3.14.0  | 3.14.0  | 2.0.12 |
| 3.14.0  | 3.23.0  | 2.0.12 |
| 3.23.0 | latest | latest |


### Upgrading from < 3.14.0 to >= 3.14.0

In leihs 3.15.0, we fixed a lot of potential database inconsistency problems. An inconsistency can for example be that a contract exists for an inventory pool that has since been deleted. Because databases with such inconsistencies are no longer compatible with leihs 3.15.0, we introduced a report page in leihs 3.14.0 that lets you look at your inconsistent data and decide what to do with it. leihs cannot decide automatically, a human with good knowledge about the inventory in question is required.

So if you have a leihs database from a leihs version older than 3.14.0, you will first have to upgrade to 3.14.0 before continuing. This is easily done checking out version 3.5.0 as described above under "Performing Upgrades", then logging into your leihs 3.14.0 instance as an administrator. Navigate to "Database Check" -> "Consistency" (the direct path is `/admin/database/consistency`.

Each block in this report is labelled with the consistency problem it represents, for example "3 groups with missing inventory_pools". Looking at the report, you should be able to decide whether deleting any of these items is safe. If you want to delete an item directly from this screen, there is a big red "Delete from the database" button there for you to use.

You also see the SQL statement that discovered these inconsistencies, for example:

```sql
SELECT `groups`.* FROM `groups` LEFT JOIN inventory_pools AS t2 ON groups.inventory_pool_id = t2.id WHERE (`groups`.`inventory_pool_id` IS NOT NULL) AND `t2`.`id` IS NULL
```
You may use these statements as a basis for deleting the offending data, if you prefer to use SQL.

Once all the problems have been cleared, the report will be empty. You can now upgrade beyond version 3.14.0 to 3.15.0 or higher. 3.15.0 is the version that introduces foreign key checks so that inconsistencies like these are impossible.


### Upgrading from < 3.5.0 to >= 3.5.0

If you have a version older than 3.5.0, you will first have to upgrade to 3.5.0 in order to get the database into a state compatible with versions higher than 3.5.0. This is easily done checking out version 3.5.0 as described above under "Performing Upgrades", then checking out the version you want and upgrading to that.

Please note that to upgrade to 3.5.0, you need an older version of Rubygems since leihs 3.5.0 uses an old version of the acts-as-dag gem, whose gemspec is not compatible with newer version of Rubygems. Therefore you must first downgrade your Rubygems version:

```gem update --system 2.0.14```

Once you have upgraded to leihs 3.14.0 and beyond, you can freely move to a newer version or Rubygems as well.


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
