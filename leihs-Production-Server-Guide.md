# leihs Production Server Guide


This guide has become obsolete.

The Rails landscape changes faster than we can upgrade this guide, but here are is how you can host leihs in a few keywords:

  * Use [Debian GNU/Linux](http://www.debian.org) 8 (jessie) or later.
  * Use the Apache Web server.
  * Install the [Phusion Passenger](http://modrails.com) packages.
  * Use [rbenv](https://github.com/sstephenson/rbenv) so you can host Ruby 2.3 applications.
  * Install Ruby 2.3 and then Bundler: ```gem install bundler```
  * Install leihs in a good location, then ```bundle install --deployment --without development test```
  * Follow the database and LDAP setup steps from the [[leihs-Admin-Guide]].
  * Create a virtual host for your leihs instance and point your Apache at that.

That should get you a nice production setup. leihs handles similarly to any other Rails application, so anything you know about hosting Rails applications applies here as well.
