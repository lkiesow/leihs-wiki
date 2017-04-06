Almost all Leihs upgrades work the same way:

* Put the new leihs source code somewhere. For example
  * new directory: /home/leihs/leihs-N.N.N
  * old directory: /home/leihs/leihs-O.O.O
* Copy the following things from your old leihs installation to your new installation:

        cd /home/leihs/leihs-O.O.O
        cp -a -r ./public/images/attachments ../leihs-N.N.N/public/images/
        cp -a -r ./public/attachments ../leihs-N.N.N/public/
        cp -a ./config/database.yml ../leihs-N.N.N/config/
        cp -a ./config/initializers/secret_token.rb ../leihs-N.N.N/config/initializers/
        cp -a ./config/LDAP.yml ../leihs-N.N.N/config/

* Create and set permissions to directories needing write-access. Use the same commands as during installation. See "Create the temporary directories that are necessary for e.g. image uploads". Do not forget the log file directory.
* Run in the new leihs directory: `bundler install` to upgrade your gems and tools the new version needs.
* Run in the new leihs directory: `RAILS_ENV=production bundle exec rake db:migrate`
* Run in the new leihs directory: `RAILS_ENV=production bundle exec rake assets:precompile`
* Restart your leihs server (how you do this depends on how you're running leihs, whether standalone, through Phusion Passenger, etc.)
* Change the hardcoded path to the new Leihs directory inside your cronjob shellscript

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
 * 3.23.0 to 3.36.0
 * 3.36.0 to the latest version

It's necessary to use a specific (and outdated) version of Rubygems to be able to install all the gems for specific (and also outdated) versions of leihs. The following table shows which Rubygems version is needed at which point:

| from leihs |to leihs  | Rubygems version |
|---|---|---|
|< 3.5.0  | 3.5.0 | 2.0.12  |
|< 3.14.0  | 3.14.0  | 2.0.12 |
| 3.14.0  | 3.23.0  | 2.0.12 |
| 3.23.0 | 3.36.0 | latest |
| 3.36.0 | latest | latest |

## Upgrading 2.9.14 to 3.0.0

Detailed instructions are to be found [here](https://github.com/leihs/leihs/wiki/Upgrading-2.9.14-to-3.0.0)

## Upgrading >= 3.0.0 to 3.45.1

Detailed instructions are to be found [here](https://github.com/leihs/leihs/wiki/Upgrading-3.0.0-to-3.45.1)

## Upgrading 3.45.1 to >= 4.0.0

Detailed instructions are to be found [here](https://github.com/leihs/leihs/wiki/Deployment#upgrading-from-v3-to-v4)
