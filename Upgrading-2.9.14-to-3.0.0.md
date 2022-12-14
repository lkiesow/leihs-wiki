# Preparing your server

You will need to have the following on your server:

* Ruby 2.1.5
* ImageMagick
* Compilers: gcc and g++
* MariaDB or, if you can't get to that, MySQL
* Apache and [Phusion Passenger](http://www.modrails.com).


## Upgrading your system to Debian GNU/Linux 7.0 (wheezy)

Our old guides were written for Debian GNU/Linux 6.0 (squeeze). If you have not done so already, please upgrade your base system to Debian GNU/Linux 7.0 as it already has most of the dependencies available. Upgrading from Debian 6.0 to 7.0 is covered in several upgrade guides, try [this guide](http://www.garron.me/en/linux/upgrade-debian-squeeze-wheezy-6.0-7.0.html).


## Installing the base packages

The following should be enough:

    apt-get install build-essential make libxslt-dev libxml2-dev curl make build-essential git libxslt-dev libreadline6-dev libssl-dev libyaml-dev autoconf libgdbm-dev libncurses5-dev automake libtool bison libffi-dev imagemagick libcurl4-openssl-dev libmagickwand-dev apache2-prefork-dev apache2 apache2-worker-mpm

## Installing MariaDB packages

We recommend not using MySQL because MySQL development is moving in a direction that is hostile to Free Software (see "History" section in the relevant [Wikipedia article](https://en.wikipedia.org/wiki/MySQL#History).

Instead, we strongly recommend you use MariaDB if the well-being of the Free Software ecosystem is important to you. Please see the [MariaDB web page](https://mariadb.org) for installation instructions for your distribution.

If you're on Debian GNU/Linux 7, you can follow [this installation guide](http://www.x2q.net/blog/2013/05/05/howto-install-mariadb-on-debian-7-slash-wheezy/).

Also make sure to install libmariadbclient-dev:

        # apt-get install libmariadbclient-dev

## Installing Ruby 2.1.5

You will need a Ruby version manager in order to get Ruby 2.1.5 onto your system. We recommend rbenv.

**Note that all the following steps are meant to be carried out using Ruby 2.1.5, not any older Ruby version.**

## Upgrading Bundler and Rubygems

Upgrade Rubygems in case you haven't done that in a while:

    # gem update --system

Install the latest version of Bundler:

    # gem install bundler

# Retrieving the leihs 3 source code

This step depends a bit on how you've installed leihs 2.9 (from git or from a tarball). We assume that you've installed from a tarball, for example to `/home/leihs2.9` and you're going to install 3.0 in `/home/leihs-3.0.2`.

        # cd /home
        # wget https://github.com/zhdk/leihs/archive/3.0.2.tar.gz
        # tar xvfz 3.0.2.tar.gz
        # chown -R leihs /home/leihs-3.0.2

Note that we assume you have a user `leihs` that you ran leihs 2.9 as, and that you also want to use for leihs 3.0.

This will give you a new directory `/home/leihs-3.0.2`. We'll assume that you install and run leihs from there in future. If you use an automated deployment system such as [Capistrano](http://www.capistranorb.com) (which we recommend), the target directory will probably be somewhere else, and will be managed automatically by Capistrano.

But let's continue to imagine we'll put leihs 3.0 in `/home/leihs-3.0.2`, it's easier to illustrate the upgrade process that way.


# Migrating your existing configuration

You'll have to retrieve some things from your old leihs 2.9 installation:

* Database configuration.
* Any customization you have made to `config/environment.rb`
* Any customization you have made to controllers, such as the LDAP authentication controller.
* Image and file attachments.

## Database configuration

First things first, you will have to copy the username, password, host, port and database name from your old `config/database.yml`. Do *not* copy the `adapter:` line. Instead, you must change the adapter to `mysql2`. The `mysql` adapter only existed for Ruby 1.8, it can't be used on Ruby 1.9 applications. Here is an example that works:


        production:
           adapter: mysql2
           database: leihs_production
           encoding: utf8
           username: root
           password:
           host: localhost
           port: 3306

Use the same database as for your leihs 2.9 installation. We will later upgrade your existing data and schema to fit leihs 3.0.

We use the `mysql2` adapter even though we're connecting to MariaDB. MariaDB is a drop-in replacement for MySQL and as such, the connectors are usually compatible.


## environment.rb customizations

If you have changed any of the constants in `config/environment.rb` in your leihs 2.9 installation, you will have to transfer these settings to the new Settings area in the admin section.

Examples for these settings are:


        LOCAL_CURRENCY_STRING = "GBP"
        CONTRACT_TERMS = "Don't break anything, please"
        CONTRACT_LENDING_PARTY_STRING = "Your\nAddress\nHere"
        EMAIL_SIGNATURE = "Das PZ-leihs Team"
        LDAP_CONFIG = YAML::load_file("#{Rails.root}/config/LDAP.yml")
        DEFAULT_EMAIL = 'sender@example.com'
        DELIVER_ORDER_NOTIFICATIONS = false
        USER_IMAGE_URL = "http://www.zhdk.ch/?person/foto&width=100&compressionlevel=0&id={:id}"

Copy these settings to some safe place so you can copy/paste them into your browser later. *Do not* use a `config/environment.rb` file with leihs 3.0.0. This file only applies to leihs 2.9.



## Controller customizations

If you have customized any controllers, e.g. the LDAP authentication controller, you will have to port your changes to one of the LDAP controllers available in leihs 3.0. They are still in `app/controllers/authenticator` and work mostly the same way, just with Ruby 1.9 and Rails 3 syntax.


## Images and attachments

Let's copy the images and attachments to leihs 3.0. They can be used unaltered for both leihs 2 and leihs 3:

        $ cp -a /home/leihs2.9/public/images/attachments /home/leihs-3.0.0/public/images/attachments
        $ cp -a /home/leihs2.9/public/attachments /home/leihs-3.0.0/public/attachments

If you've done something else, like symlinking your images to some central location, you can of course simply link your leihs 3 attachments and image attachment directories to those locations as well. Nothing will break.


# Install the gems

        $ cd /home/leihs-3.0.0
        $ bundle install

It will take a while, but finally a whole bunch of new gems should be installed under the `leihs` user's account.


# Running the new database migrations

Now *back up your production database*. Once you're done, continue by migrating the database:

        $ cd /home/leihs-3.0.0
        $ RAILS_ENV=production bundle exec rake db:migrate

If everything went well, the migration took some time but exited without errors.


# Precompiling assets

Something that's new in Rails 3.0 and that you didn't have to do with leihs 2.9 (because it uses Rails 2.1) is precompile assets. Precompiling will generate all sorts of images, JavaScripts etc. that are necessary to display the leihs web pages.

        $ cd /home/leihs-3.0.0
        $ RAILS_ENV=production bundle exec rake assets:precompile

This can take a very long time. Don't be alarmed if it runs for more than 15 minutes. You will have to repeat this step every time you upgrade leihs 3.0, but don't worry, we will give you upgrade instructions when the time comes.


# Testing the server

You should be able to try leihs 3.0 now:

        $ cd /home/leihs-3.0.0
        $ RAILS_ENV=production bundle exec rails s -p 3000

This starts a server in production mode on port 3000. Please keep in mind that in production mode, your Rails server will not serve any assets, so if you visit your server at port 3000 in your browser, you will get a page that looks weird. You will either have to start serving static assets through this Rails server by setting `config.serve_static_assets` to `true` in `config/environments/production.rb` or you will have to install an e.g. Apache or nginx server in front of your leihs installation through Phusion Passenger.

If there were no error messages on the console or in your browser when accessing your leihs installation, you may stop the test server with Ctrl-C.

# Creating a virtual host

We recommend using [Phusion Passenger](http://www.modrails.com) and creating a virtual host for your leihs installation.

# Cleaning up after ourselves

## Removing obsolete dependencies (optional)

leihs 2.9 needed more packages and external dependencies than leihs 3.0. It also needed Ruby 1.8, which you don't need for 3.0. You can remove these unnecessary packages if you like, saving some space on your server and getting rid of dead code.

If you're not sure whether you still need any of these packages for other software, do not remove them.

### Removing Ruby 1.8.7

leihs 3.0 only uses Ruby 1.9 or higher, so if you don't want to keep leihs 2.9 around, there is no need to have Ruby 1.8:

If you've used RVM to install Ruby 1.8.7, you can remove it like so:

        $ rvm remove 1.8.7

If you've installed a Debian package for Ruby 1.8.7, you can remove it like so:

        # apt-get remove ruby1.8
        # apt-get autoremove


### Removing obsolete dependencies

The following packages are not required for leihs 3.0:

        # apt-get remove libcairo2-dev libcairo2 sphinxsearch
        # apt-get autoremove

You may also remove Node.js unless you need that for something else. How to remove Node depends on how you've installed it.


