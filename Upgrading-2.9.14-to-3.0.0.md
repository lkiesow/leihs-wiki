# Preparing your server

You will need to have the following on your server:

* Ruby 1.9.2 or newer
* ImageMagick
* Compilers: gcc and g++
* MariaDB or, if you can't get to that, MySQL

In addition to that, we recommend hosting leihs behind Apache using [Phusion Passenger](http://www.modrails.com).


## Upgrading your system to Debian GNU/Linux 7.0 (wheezy)

Our old guides were written for Debian GNU/Linux 6.0 (squeeze). If you have not done so already, please upgrade your base system to Debian GNU/Linux 7.0 as it already has most of the dependencies available. Upgrading from Debian 6.0 to 7.0 is covered in several upgrade guides, try [this guide](http://www.garron.me/en/linux/upgrade-debian-squeeze-wheezy-6.0-7.0.html).


## Installing the base packages

The following should be enough:

    apt-get install build-essential make libxslt-dev libxml2-dev curl make build-essential git libxslt-dev libreadline6-dev libssl-dev libyaml-dev autoconf libgdbm-dev libncurses5-dev automake libtool bison libffi-dev imagemagick libcurl4-openssl-dev libmagickwand-dev

## Installing MariaDB packages

We recommend not using MySQL because MySQL development is moving in a direction that is hostile to Free Software (see "History" section in the relevant [Wikipedia article](https://en.wikipedia.org/wiki/MySQL#History).

Instead, we strongly recommend you use MariaDB if the well-being of the Free Software ecosystem is important to you. Please see the [MariaDB web page](https://mariadb.org) for installation instructions for your distribution.

If you're on Debian GNU/Linux 7.0, you can follow [this installation guide](http://www.x2q.net/blog/2013/05/05/howto-install-mariadb-on-debian-7-slash-wheezy/).


## Installing a Ruby version manager (optional)

If you've followed other guides involving Ruby or have some other Rails applications installed, you probably already have [RVM](http://rvm.io). If not, we really recommend installing RVM. A combination of Phusion Passenger 4.0 and RVM will allow you to host web applications using different Ruby versions side by side, which might be useful while you are migrating from leihs 2 to leihs 3.

If you're happy with just one Ruby version per server, feel free to skip RVM.

## Installing Ruby 1.9.3

If you *do not* want to use RVM, you can install a system-wide Ruby 1.9.x version:

    # apt-get install ruby1.9.3

## Upgrading Bundler and Rubygems

You should already have Bundler installed 


# Migrating your existing configuration

# Installing Ruby 1.9.2 or newer


# Removing obsolete dependencies (optional)

## Removing Ruby 1.8.7

## Removing obsolete dependencies
