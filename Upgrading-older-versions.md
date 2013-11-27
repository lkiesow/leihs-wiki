# Upgrading from and to versions older than 2.9.14

### Upgrading from one 2.9.x version to another 

Make absolutely sure to **back up** your entire leihs installation as well as your entire leihs database before starting.

1. Download the new leihs version (in this example, 2.9.8 version is used) and unpack it to a new location:

        $ tar xvfz leihs-2.9.8.tar.gz
        $ cd leihs-2.9.8

2. Copy any images uploaded to your old leihs version so they are also available in the new leihs. 2.9.x refers to your old leihs version in this example:

        $ rm -rf public/images/attachments
        $ cp -pr ../leihs-2.9.x/public/images/attachments public/images/

3. Copy your old database configuration file:

        $ cp ../leihs-2.9.x/config/database.yml config/

4. Look at your old config/environment.rb file in a text editor. Open the new config/environment.rb. Copy the configuration items from your old config to your new config as necessary. You might find new options in the new environment.rb that weren't present in the old one. Save your new configuration.

5. Install bundler and update rake as root:

        # gem install bundler
        # gem install rake

6. Run the database migration to get all your data up to the new version:

        $ RAILS_ENV=production rake db:migrate

7. Run the server. Make sure you stop your previous leihs server. If you use mod_passenger, do `touch tmp/restart.txt` instead.

        $ RAILS_ENV=production bundle exec ./script/server

8. Stop the Sphinx server that's running for leihs 2.9.x.:

        $ cd leihs-2.9.x
        $ RAILS_ENV=production rake ts:stop

9. Reindex Sphinx at the new location and restart the Sphinx server: 

        $ cd leihs-2.9.8
        $ RAILS_ENV=production bundle exec rake ts:config
        $ RAILS_ENV=production bundle exec rake ts:reindex
        $ RAILS_ENV=production bundle exec rake ts:start

If everything went correctly, you should see leihs coming up at http://localhost:3000 or, if using mod_passenger, at the location you configured. 


### Upgrading from leihs 2.0 to leihs 2.1 

The upgrade should be painless.

1. Download the new leihs version (in this example, the .tar.gz version is used) and unpack it to a new location:

        $ tar xvfz leihs-2.1.tar.gz
        $ cd leihs-2.1

2. Copy any images uploaded to your old leihs version so they are also available in the new leihs:

        $ rm -rf public/images/attachments
        $ cp -pr ../leihs-2.0/public/images/attachments public/images/

3. Copy your old database configuration file:

        $ cp ../leihs-2.0/config/database.yml config/

4. Look at your old config/environment.rb file in a text editor. Open the new config/environment.rb. Copy the configuration items from your old config to your new config as necessary. You might find new options in the new environment.rb that weren't present in the old one. Save your new configuration.

5. Run the database migration to get all your data up to the new version:

        $ RAILS_ENV=production rake db:migrate

6. Rerun the steps from the section "Configure and start the Sphinx server" relevant for your operating system in order to create/update your fulltext search index and make sure Sphinx is running.

7. Run the server. Make sure you stop your previous leihs server!

        $ RAILS_ENV=production ./script/server

If everything went correctly, you should see leihs coming up at http://localhost:3000. 



### Upgrading from leihs 2.1 to leihs 2.2 

The upgrade should be painless.

1. Download the new leihs version (in this example, the .tar.gz version is used) and unpack it to a new location:

        $ tar xvfz leihs-2.2.tar.gz
        $ cd leihs-2.2

2. Copy any images uploaded to your old leihs version so they are also available in the new leihs:

        $ rm -rf public/images/attachments
        $ cp -pr ../leihs-2.1/public/images/attachments public/images/

3. Copy your old database configuration file:

        $ cp ../leihs-2.1/config/database.yml config/

4. Look at your old config/environment.rb file in a text editor. Open the new config/environment.rb. Copy the configuration items from your old config to your new config as necessary. You might find new options in the new environment.rb that weren't present in the old one. Save your new configuration.

5. Run the database migration to get all your data up to the new version:

        $ RAILS_ENV=production rake db:migrate

6. Run the server. Make sure you stop your previous leihs server!

        $ RAILS_ENV=production ./script/server

If everything went correctly, you should see leihs coming up at http://localhost:3000. 


### Upgrading from leihs 2.2 to leihs 2.9 

WARNING: You must upgrade from 2.2.x to 2.9.1 before continuing on with the 2.9 series. The reason is that with 2.9, we rolled up all previous migrations into one single file to make future migrations faster and easier. The system will warn you if you try to upgrade beyond 2.9.1 without installing 2.9.1 first. The sequence of actions is the same for all 2.9 releases, so once you have 2.9.1, you can just grab e.g. 2.9.4 and follow these instructions again.

The upgrade should be painless, but please note that the access levels were replaced by user groups in this release. This can give you almost the same behavior, but it's different to manage. You now specify groups in the admin interface or your inventory pool's backend, and then specify how many items of each model that each group can have.

If you had existing user levels in your system, these will be automatically converted to user groups during the database migration (db:migrate). This is documented below.

1. Download the new leihs version (in this example, the .tar.gz version is used) and unpack it to a new location:

        $ tar xvfz leihs-2.9.tar.gz
        $ cd leihs-2.9

2. Copy any images uploaded to your old leihs version so they are also available in the new leihs:

        $ rm -rf public/images/attachments
        $ cp -pr ../leihs-2.2/public/images/attachments public/images/

3. Copy your old database configuration file:

        $ cp ../leihs-2.2/config/database.yml config/

4. Look at your old config/environment.rb file in a text editor. Open the new config/environment.rb. Copy the configuration items from your old config to your new config as necessary. You might find new options in the new environment.rb that weren't present in the old one. Save your new configuration.

5. Install bundler and update rake as root:

        # gem install bundler
        # gem install rake

6. Run the database migration to get all your data up to the new version:

        $ RAILS_ENV=production rake db:migrate

7. Run the server. Make sure you stop your previous leihs server! If you use mod_passenger, do `touch tmp/restart.txt` instead.

        $ RAILS_ENV=production bundle exec ./script/server

If everything went correctly, you should see leihs coming up at http://localhost:3000 or, if using mod_passenger, at the location you configured. 

