# leihs Production Server Guide 
Ramon Cahenzli <ramon.cahenzli@zhdk.ch>

## Introduction

We don't want to prescribe any specific way for installing leihs in a production environment. Your server landscape is certainly not the same as ours, and we don't want to force any specific architecture on you just for running leihs.

What we can provide here, however, is an example of a completely suitable production server for leihs, including automated deployments and upgrades. You can take this example and customize it in any way that fits your server landscape.

We will use these components:

  * Debian GNU/Linux or a RedHat-based GNU/Linux distribution as operating system
  * Capistrano for automated deployment
  * Phusion Passenger to manage the Rails application server pool
  * Apache to provide a web server
  * Apache's mod_proxy if you want to run leihs 2.9 and 3.0 in parallel
  * Passenger Standalone if you want to run leihs 2.9 and 3.0 in parallel
  * MySQL as database server

In the end, you will have a complete leihs production environment.


## Base system
### Installing the base components

Follow the instructions from the leihs Admin Guide for your platform (Debian or RPM-based distribution). *But instead of installing Ruby from your package manager in step 1, substitute this new step 1*.

For Debian-based distros:

        # apt-get install curl make build-essential git libxslt-dev libcairo2-dev libmysqlclient-dev libxml2-dev mysql-server

For RPM-based distros:

        # yum install curl make git wget gcc gcc-c++ libreadline-devel openssl-devel libxslt-devel libxml2-devel libxml2 mysql-devel cairo-devel mysql-server

Continue following the rest of the distribution-specific instructions from step 2 on until you get to the section "Installing the platform-independent components". Instead of following the Admin Guide, follow these steps:

## Installing the platform-independent components in a production situation

Download and install Sphinx (a fulltext search system). In this example we also include libstemmer, a library that allows for word stem searching in various languages. We use version 0.9.9: 

        # mkdir -p /root/software
        # cd /root/software
        # wget http://sphinxsearch.com/downloads/sphinx-0.9.9.tar.gz
        # tar xvfz sphinx-0.9.9.tar.gz
        # cd sphinx-0.9.9
        # wget http://snowball.tartarus.org/dist/libstemmer_c.tgz
        # tar xvfz libstemmer_c.tgz
        # ./configure --with-libstemmer && make
        # make install

Download and install Node.js, a JavaScript runtime. You can find the latest version at http://nodejs.org.

        # cd /root/software
        # wget http://nodejs.org/dist/v0.8.20/node-v0.8.20.tar.gz
        # tar xvfz node-v0.8.20.tar.gz
        # cd node-v0.8.20
        # ./configure
        # make
        # make install


### Installing Ruby and related software

1. Install RVM from [the RVM website](http://rvm.io). We can use RVM to install different Ruby versions on demand, to upgrade or downgrade between Ruby versions and to run Ruby 1.8 and Ruby 1.9 applications on the same server.

The RVM install process is usually aided by a simple Bash script you can execute directly. We recommend installing as root, which will create a system-wide RVM installation instead of a per-user RVM installation:

        # curl -L https://get.rvm.io | bash -s stable --ruby 

Reload your shell to make sure RVM is loaded:

        # bash --login

This will install RVM as well as the latest stable Ruby version and RubyGems, a management system for Ruby libraries.

Now install the Bundler gem, a Ruby dependency manager:

        # gem install bundler

Install Ruby 1.8.7 if you want to use leihs 2.9 in parallel with leihs 3.0. If you only want to use 3.0 or higher, this is not necessary:

        # rvm install 1.8.7
        # rvm use 1.8.7
        # gem install bundler

Revert to using Ruby 1.9.3 once you're done (we'll be using Ruby 1.9 for Capistrano later on):

        # rvm use 1.9.3

### Setting up Capistrano

Capistrano is a deployment system that automates deployments for you. You write Capistrano recipes, and Capistrano then executes those recipes against your production server via SSH.

If you already have Capistrano, this chapter won't be news to you. If not, you need to set up Capistrano on some machine that can reach the deployment servers.

If you have your own local machine that can run Ruby and SSH, you can install Capistrano on that machine. You can also install Capistrano on the target server itself and then deploy from the target server to the target server, in case you do not wish to install Ruby on any local workstations.

The idea is this: You install Capistrano on an SSH-capable machine, check out the leihs source code to that machine and run the Capistrano deployment recipes included with leihs. You will write your own copies of those recipes, adapted for your production servers, so that the recipes deploy leihs on the actual production servers.

In the following example, we will install Capistrano on the production server itself and then deploy from the server to itself.

        # gem install capistrano capistrano-ext rvm-capistrano

### Getting the leihs source code

Prepare a directory for the leihs source code to live in and clone the leihs git repository to it:

        # cd /root/software
        # git clone git://github.com/zhdk/leihs.git
        # cd leihs
        # git checkout next

You will be working on the "next" branch for now, the latest development code.

### Creating a new deployment recipe for your own servers

You will be writing a new deployment recipe in `config/deploy`. You can copy an existing recipe to your new file, and we will start with a staging recipe (meant to deploy onto a test instance) instead of a production recipe (meant for your production server):

        # cd leihs
        # cp config/deploy/staging.rb config/deploy/staging-myserver.rb

At this point, it would be good to learn about git so you can check your recipes into version control. The git link above points to a read-only copy of our git repository for leihs. You can either fork your own on GitHub and add a remote that points there, or you can create a git repository for leihs somewhere else in your organization.

Describing how to set up a git environment for your use goes far beyond this server guide, please refer to the [http://git-scm.com/book](Git book) for more information.

Open `config/deploy/staging-myserver.rb` in your favorite text editor. The file is written in the Capistrano domain-specific language (DSL). You can also use Ruby in most of the tasks.

At the very least, you will have to adapt the following configuration settings:

 * `rvm_ruby_string`: The version of Ruby to use on the target server for running this copy of leihs.
 * `application`: The application name. Mostly for your reference, unless you want to use this variable later on in the configuration.
 * `db_config`: The location on the target server of the `database.yml` file, containing database credentials for this instance of leihs. We recommend putting this in the base directory of the user you want to run this instance of leihs as, e.g. in /home/leihs-test/database.yml.
 * `app_config`: The location of the `application.rb` file on the target server. This file is needed to define a few constants and global settings that cannot be set from inside the database.
 * `ldap_config`: The location of the `LDAP.yml` file containing LDAP binding credentials, if you want to use LDAP authentication.
 * `deploy_to`: The directory on the target server where this instance will be deployed to. For example: `/home/leihs-test` as a home for your staging instance. Capistrano will create several directories under this location.
 * `app`, `web`, and `db`: For each server role, you could provide separate physical servers. In our example, we will deploy all three roles on the same server. Use an "SSH syntax" for this, e.g. leihs-test@localhost means the instance will be deployed as user leihs-test on localhost.

Before you make your first use of this configuration, you need to have a user to run the instance as. Going with the examples above, you could use `leihs-test`. Add the user on your server:

        # adduser leihs-test
        # passwd leihs-test

We recommend using SSH public key authentication, but this guide will not explain how to set this up.

Now you are ready to run Capistrano for the first time to set up its directories. From the leihs source code directory, do this:

        # cap staging-myserver deploy

You will notice that new directories appeared under /home/leihs-test: `releases` and `shared`. `releases` is where the cloned source code of leihs will appear once you deploy. `shared` is where Capistrano places files that are shared between all deployments of this particular instance, for example images, attachments, user uploads, etc.

Next you need to create a database configuration file in the location specified under the `db_config` variable in your deployment recipe. Here is an example:

        production:
           adapter: mysql2
           database: leihs_staging
           encoding: utf8
           username: root
           password: root
           host: localhost
           port: 3306

Next, create an LDAP configuration file in the location specified in `ldap_config`, if you wish to use LDAP authentication. Here is an example:

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

Finally, copy config/application.rb from the leihs source code to your instance:

        # cp /root/leihs/config/application.rb /home/leihs-test/application.rb

Open the file in your favorite text editor. You can change some constants in this file if necessary: LOCAL_CURRENCY_STRING, CONTRACT_TERMS, CONTRACT_LENDING_STRING, EMAIL_SIGNATURE, DEFAULT_EMAIL, DELIVER_ORDER_NOTIFICATIONS, USER_IMAGE_URL and PER_PAGE.

Please note that some of these variables can be customized via the deployment recipe on every deploy as well, so you might not need to change them here. Instead, they could be changed using sed in `config/deploy/staging-myserver.rb`.

Now it's time to try our recipe in earnest (well, almost). A cold deploy, meaning that all the steps that are normally taken during deploy will be taken, except for actually starting the application.

        # cd /root/software/leihs
        # cap staging-myserver deploy:cold

If all of this worked, it means you have tackled all the largest hurdles already. The rest is just a matter of configuration and installing some more gems and extensions.


### Installing Phusion Passenger and creating a Rails virtual host

The following host will host leihs 3.x using Ruby 1.9.x. This is in contrast to leihs 2.9.x, which needs to be run on Ruby 1.8.7 and will be handled differently later on.

TODO

5. Create any temporary directories that are necessary for e.g. image uploads, temporary files etc. Make sure to create these directories so that the leihs user has write permission to them.

        $ cd /home/leihs
        $ mkdir -p public/images/attachments
        $ mkdir -p tmp/sessions
        $ mkdir tmp/cache
        $ mkdir -p log
 
6. Configure and start the Sphinx server:

        $ cd /home/leihs
        $ RAILS_ENV=production bundle exec rake ts:config
        $ RAILS_ENV=production bundle exec rake ts:reindex
        $ RAILS_ENV=production bundle exec rake ts:start


7. Set up a system cronjob that sends nightly e-mail reminders and, more importantly, updates all the models' availability counts. There are many ways to schedule repeating tasks on GNU/Linux, but here's a line in crontab-format that you can add to your leihs user's crontab using e.g. `crontab -e`:

        1 00    * * *   cd /home/leihs && RAILS_ENV=production rake leihs:cron

    The important bit here is to run the "leihs:cron" rake task. How you do this exactly is irrelevant.



## Users, logins and levels 

### Default admin username/password 

After installation, a default user is created for the Database Authentication module. Username: super_user_1. Password: pass.

Please change the super_user_1 password immediately after logging in the first time. Otherwise other people will also be able to log in using the well known default password.


### Hooking up to LDAP for logins 

Currently, leihs contains only a very rudimentary LDAP login adapter. It was developed by the Zurich University of the Arts specifically for the Hochschule der Künste Bern, Switzerland, and some parts of it are hardcoded to fit the environment there.

However, it's not impossible to get leihs to authenticate against your own LDAP server if you are willing to modify two files in leihs.

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

    $ RAILS_ENV=production bundle exec ./script/console

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

