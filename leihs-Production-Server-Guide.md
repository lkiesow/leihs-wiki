# leihs Production Server Guide 
Ramon Cahenzli <ramon.cahenzli@zhdk.ch>

## Introduction

We don't want to presrcibe any specific way for installing leihs in a production environment. Your server landscape is certainly not the same as ours, and we don't want to force any specific architecture on you just for running leihs.

What we can provide here, however, is an example of a completely suitable production server for leihs, including automated deployments and upgrades. You can take this example and customize it in any way that fits your server landscape.

We will use these components:

  * Debian GNU/Linux or a RedHat-based GNU/Linux distribution as operating system
  * Capistrano for automated deployment
  * Phusion Passenger to manage the Rails application server pool
  * Apache to provide a web server
  * Apache's mod_proxy if you want to run leihs 2.9 and 3.0 in parallel
  * MySQL as database server

In the end, you will have a complete leihs production environment that can run leihs 2.x and 3.x.


## Base system
### Installing the base components using Debian GNU/Linux

These instructions were tested on a minimal install of Debian GNU/Linux 5.0 (Lenny) and Debian GNU/Linux 6.0 (Squeeze). They might also work on Ubuntu. You might have to substitute `sudo su` for `su` because Ubuntu does not configure a root password, thus `su` would not work.

1. Install libxslt, Cairo, MySQL client libraries, libxml2:

        # apt-get install libxslt-dev libcairo2-dev libmysqlclient-dev libxml2-dev

2. Install the MySQL header files and the MySQL gem:

        # apt-get install libmysqlclient15-dev make build-essential
        # gem install mysql

3. Install ImageMagick:

        # apt-get install imagemagick libmagickwand-dev

4. Optional: Speed boost thanks to memcached

    You can install memcached in order to make leihs perform faster, especially for activities that require recalculations of item availability. Memcached can speed up the system by several orders of magnitude.

        # apt-get install memcached

    Don't forget to set ENABLE_MEMCACHED to "yes" in /etc/defaults/memcached

### Installing the base components on RedHat Enterprise Linux, Fedora or CentOS

The following instructions apply to most RPM-based distributions and were tested on CentOS 6.4.

1. Install libxslt, Cairo, MySQL client libraries, libxml2, gcc and some required dependencies:

        # yum install wget gcc gcc-c++ libreadline-devel openssl-devel libxslt-devel libxml2-devel libxml2 mysql-devel cairo-devel

2. Install the MySQL header files and the MySQL gem:

        # yum install mysql-server mysql gcc make ruby-devel
        # gem install mysql

3. Install ImageMagick:

        # yum install ImageMagick ImageMagick-devel 

4. Optional: Speed boost thanks to memcached

    You can install memcached in order to make leihs perform faster, especially for activities that require recalculations of item availability. Memcached can speed up the system by several orders of magnitude.

        # yum install memcached


## Installing Sphinx

Download and install Sphinx (a fulltext search system). In this example we also include libstemmer, a library that allows for word stem searching in various languages. We use version 0.9.9: 

        $ cd /tmp
        $ wget http://sphinxsearch.com/downloads/sphinx-0.9.9.tar.gz
        $ tar xvfz sphinx-0.9.9.tar.gz
        $ cd sphinx-0.9.9
        $ wget http://snowball.tartarus.org/dist/libstemmer_c.tgz
        $ tar xvfz libstemmer_c.tgz
        $ ./configure --with-libstemmer && make
        $ su
        # make install


## Installing Ruby and related software

1. Install RVM from [the RVM website](http://rvm.io). We can use RVM to install different Ruby versions on demand, to upgrade or downgrade between Ruby versions and to run Ruby 1.8 and Ruby 1.9 applications on the same server.

2. Install RubyGems from [the RubyGems website](http://rubygems.org/). Make sure *not to install* the edition of RubyGems that is available from Debian's package archives. RubyGem development moves so quickly that we need to use the one from upstream.

        # cd /tmp
        # wget http://production.cf.rubygems.org/rubygems/rubygems-1.5.2.tgz 
        # tar xvfz rubygems-1.5.2.tgz
        # cd rubygems-1.5.2
        # ruby setup.rb
        # ln -s /usr/bin/gem1.8 /usr/bin/gem

Note that the URL above might change! Please visit the RubyGems site to find the exact URL under "Downloads". It is important that you use version 1.5.2 or 1.5.3 or RubyGems because newer versions don't work with Rails 2.3.5, which leihs uses.


3. Configure database access for this installation of leihs. Copy the file config/database.yml.example to config/database.yml and set things up according to your needs. You will need a MySQL database for leihs. Here is an example of a production database configuration:

        production:
           adapter: mysql
           database: leihs_production
           encoding: utf8
           username: root
           password:
           host: localhost
           port: 3306

4. Create and migrate the database:

        # su - leihs
        $ RAILS_ENV=production bundle exec rake db:migrate
        $ RAILS_ENV=production bundle exec rake db:seed

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

