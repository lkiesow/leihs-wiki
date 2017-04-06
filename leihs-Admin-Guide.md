# Users, logins and roles

## Default admin username/password

After installation, a default user is created for the Database Authentication module.

Username: admin
Password: password

Please change the admin password immediately after logging in the first time. Otherwise other people will also be able to log in using the well known default password.

If logging in does not work, you should try deleting your browser cache/cookies. Some users had issues using older versions of Firefox (32.x).

The roles themselves are described under [Roles](Roles).

## Hooking up to LDAP for logins

It is possible to use LDAP for logins. The procedure is described under [[LDAP configuration]].

# Performing upgrades

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
