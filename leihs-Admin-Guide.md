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

        # apt-get install build-essential make git libssl-dev libxslt-dev libmysqlclient-dev libxml2-dev curl imagemagick libmagickwand-dev

2. Install [RVM](http://rvm.io/) and Bundler. When you're done, install Ruby:

        # rvm install 2.1.1
        # gem install bundler


## Installation of the base system on RedHat Enterprise Linux, Fedora or CentOS

Please note that **this distribution is not officially supported**, but you are welcome to try these instructions. They were verified on CentOS 6.3.

1. Install libxslt, MySQL client libraries, libxml2, gcc and some required dependencies:

        # yum install curl gcc gcc-c++ libreadline-devel openssl-devel git libxslt-devel libxml2-devel libxml2 mysql-devel ImageMagick ImageMagick-devel

2. Install [RVM](http://rvm.io/) and Bundler, since CentOS doesn't have an up to date Ruby version as a package. When you're done, install Ruby:

        # rvm install 2.1.1
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
        $ rvm use 2.1.1
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

        source /usr/local/rvm/environments/ruby-2.1.1
        rvm use 2.1.1
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


### Hooking up to LDAP for logins 

You can hook up leihs to an LDAP server. So far, we've only tried ActiveDirectory, but at three different universities. We're reasonably sure the adapter should work for any LDAP server. If it doesn't work for yours, please submit a pull request with the changes you need.

#### Modifying config/LDAP.yml

Copy config/LDAP.yml.example to config/LDAP.yml and adapt the configuration to your own LDAP server:

        production:
          host: ldapserver.example.org
          port: 636 
          encryption: simple_tls
          log_file: log/ldap_server.log
          log_level: warn
          master_bind_dn: CN=LeihsEnumUser,OU=NonHuman,OU=users,DC=example,DC=org
          master_bind_pw: 12345
          base_dn: OU=users,DC=example,DC=org
          admin_dn: CN=LeihsAdmin,OU=NonHuman,OU=users,DC=example,DC=org
          unique_id_field: sAMAccountName
          search_field: sAMAccountName

`unique_id_field`, `master_bind_dn` and `master_bind_pw` are all required.

#### host
The hostname of your LDAP server.
Active Directory: enter your AD domain name here, not just a hostname. The domain name resolves to all available Domain Controllers - good for redundancy. In the example above the value should read
'example.org'

#### port
Active Directory: Use 636 if you want to use 'simple_tls' encryption. Use 389 if you set 'encryption' to 'none'. Open your firewall to allow either 636 TCP or 389 (UDP, TCP).

#### encryption
`encryption` can be set to "none" for unencrypted or "simple_tls" for SSL encrypted connections.
'simple_tls' uses SSL encryption for the communication with the LDAP server. Strongly recommended, as passwords will be sent in plaintext over the network using'none'. Beware: your Active Directory Server needs a certificate for SSL to work. (Note: need to test the specifics of this, will update documentation later, DBR)

#### master_bind
`master_bind_dn` and `master_bind_pw` are the credentials for an LDAP user that has permission to query enough of the LDAP tree so that it can find all the users you want to grant entry to leihs to. -> Create a new user in your Active Directory, for example "LeihsEnumUser". Mind your password policy: if the password changes in AD automatically, you will not be able to log in to Leihs anymore. Just update the password in this case to the new value.
The value of 'master_bind_dn' should equal the LDAP 'distinguishedName' attribute of your 'LeihsEnumUser' user.

To copy the whole distinguishedName out of the Active Directory console (for convenience):
* Open the 'Active Directory Users and Computers' console
* In the view menu enable "Advanced Features"
* Open the properties of your new user and look for the "Attribute-Editor" register
* Copy the contents of the attribute 'distinguishedName' as the value of 'master_bind_dn'

#### base_dn
The Ldap-tree that is searched for usernames. Active Directory: Copy the 'distinguishedName' of the top Organization Unit your users are stored in. The search will look in all subdirectories of this OU for a user with an attribute you specify in 'search_field'.

#### unique_id_field
`unique_id_field` is any field in your LDAP that contains a completely unique ID for the user in question. This can be the same as `search_field` if you are sure that it's unique. Alternative: Active Directory by default creates the 'objectGUID' attribute for every user which is ideal for this purpose (just looks uglier in the database, as it is a random string of hex-values).

#### search_field
The `search_field` dictates what users will have to write in the "Login" field on login.
You may have to adapt the `search_field` option to point at the LDAP attribute that contains your usernames. In Active Directory this should normally be the 'sAMAccountName' attribute, which is the 'User-Logon-Name (pre Windows 2000)' in the GUI.

#### LDAP Library
Leihs uses the GEM net-ldap for connectivity to LDAP (as of v3.16). [https://github.com/ruby-ldap/ruby-net-ldap](https://github.com/ruby-ldap/ruby-net-ldap)
It is a port of the perl library Net::LDAP. More detailed information about the parameters can be found in the sourcecode / documentation. [http://search.cpan.org/~marschap/perl-ldap/lib/Net/LDAP.pod](http://search.cpan.org/~marschap/perl-ldap/lib/Net/LDAP.pod) 

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

Warning: Please make sure that your Rails application server has SSL enabled before you put this configuration into production. You don't want to send passwords unencrypted over the web using plain HTTP.

## Performing upgrades

Almost all leihs upgrades work the same way:

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
        $ git checkout n.n.n
        $ RAILS_ENV=production bundle exec rake db:migrate

But again, this only works if you've checked out leihs straight from git before. This won't work on a leihs installation unpacked from a tarball.


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

If you have a version older than 3.5.0, you will first have to upgrade to 3.5.0 in order to get the database into a state compatible with versions higher than 3.5.0. This is easily done checking out version 3.5.0 as described above under "Performing Upgrades", then checking out the version you want.


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
