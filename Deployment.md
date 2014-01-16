A typical production deployment process for leihs could look like the following. This is modeled after the process that is actually in use at the Zürich University of the Arts (ZHdK).

Server landscape, the servers you're deploying to:

* rails.zhdk.ch with the following:
  * Apache
  * Phusion Passenger
  * RVM
  * Ruby 1.9.3-p448 installed through RVM

* db.zhdk.ch with the following:
  * MariaDB


Client requirements:

You need to have at least Ruby 1.9.3 and the following gems on the workstation that you will be deploying from:

* capistrano
* capistrano-ext
* rvm-capistrano


## Steps to take to deploy a new version of leihs at the ZHdK

With Capistrano installed on your workstation, enter the leihs source code directory and do:

    # cap production deploy

If you have permission to log in as user `leihs@rails.zhdk.ch`, Capistrano will magically deploy a version of leihs to rails.zhdk.ch. Where does Capistrano get those usernames and server names from? How does it know what to do? The following explains exactly what happens here and in which steps.


## Rough overview of the deployment steps

When we run Capistrano, it does the following:

* Connects to the target server by SSH.
* Switches to the correct Ruby version using RVM.
* Grabs the freshest source code.
* Links some config files into the `current` release directory.
* Runs bundler to install RubyGems.
* Migrates the database to the latest version.
* Precompiles all assets.
* Touches the `/tmp/restart.txt` file to signal Apache/Passenger to restart the application.

There are some details in each of these steps. They are explained below.

## Explanation of the files in config/deploy

Most of the Capistrano magic happens in the files in config/deploy. There are several of these:

### config/deploy.rb

This file is absolutely empty. This is a remnant of the capistrano-ext extensions for multi-stage deployments. You can put some toplevel configuration in here, but we don't really need that. All serious configuration is done inside the multistage config files that we'll explain next.


### config/deploy/production.rb

The production configuration file. Production here refers to the production server, as in not the staging server. Since this guide is about deploying to production, we'll look at this first.

The first section is about RVM and how it's installed on the server. Since the server uses Phusion Passenger, the only supported Ruby version manager is RVM (in case you were wondering why this isn't using rbenv, chruby or one of the others). The options set which version of Ruby to use on the target server, and where RVM is installed. RVM is installed system-wide by the root user, which by default puts it in `/usr/local/rvm`:

```ruby
require "rvm/capistrano"
set :rvm_type, :system
set :rvm_ruby_string, '1.9.3'
set :rvm_path, "/usr/local/rvm"
```

Then there is a Capistrano extension that takes care of running `bundle` with the right options on the target host:

```ruby
require "bundler/capistrano"
```

This saves us from creating our own steps to run `bundle install` etc. What follows now is a section with a bunch of settings:


```ruby
set :application, "leihs-new"
set :scm, :git
set :repository,  "git://github.com/zhdk/leihs.git"
load 'config/deploy/stable_version'
set :deploy_via, :remote_cache
set :db_config, "/home/leihs/#{application}/database.yml"
set :ldap_config, "/home/leihs/#{application}/LDAP.yml"
set :use_sudo, false
set :rails_env, "production"
```

Most of it should be self-explanatory, but let's look at some of the more specific options. 

`:application` sets the application name, this just saves us some typing when this variable is used in paths further down. It also helps prevent clashes between two applications on the same host by making sure that each application, as long as it has a unique name in its `config/deploy/production.rb`, gets deployed to a different directory.

`load 'config/deploy/stable_version'` is a bit special: This loads the `branch` setting from an external file, telling Capistrano which branch to deploy from. By having it in a separate file, we make sure that only the version accepted by the product owner is considered stable, so that only this is deployed to production. Having it in a separate file means that every developer on the project, if they use the latest version of the deployment stuff from the master branch, will deploy the correct version.

The value of the `branch` setting can be a tag, a commit or a branch name. If it's a branch name, Capistrano will deploy the newest commit from that branch.

The configuration file paths for `:db_config` and `:ldap_config` are valid on the target host, not your own host. To preven database passwords from leaking into the source tree or elsewhere, the true credentials are only stored in `/home/leihs/leihs-new/database.yml` on the server. In the `link_config` step, Capistrano will link from this file to the current release: `/home/leihs/leihs-new/current/config/database.yml`.


The next line is there so that Capistrano doesn't create a subshell/shell wrapper when executing commands:

```ruby
default_run_options[:shell] = false
```

The next few lines are very important. They tell Capistrano which host to connect to and where to deploy to:

```ruby
set :deploy_to, "/home/leihs/#{application}"

role :app, "leihs@rails.zhdk.ch"
role :web, "leihs@rails.zhdk.ch"
role :db,  "leihs@rails.zhdk.ch", :primary => true
```

Note that the "db" role is carried out by the same host as the app and web roles. This is intentional. Database-related actions are performed on that host and the recipe does nothing that's specific to the DB server at all.


```ruby
load 'config/deploy/recipes/retrieve_db_config'
load 'config/deploy/recipes/link_config'
load 'config/deploy/recipes/link_attachments'
load 'config/deploy/recipes/link_db_backups'
load 'config/deploy/recipes/make_tmp'
load 'config/deploy/recipes/chmod_tmp'
load 'config/deploy/recipes/migrate_database'
load 'config/deploy/recipes/bundle_install'
load 'config/deploy/recipes/precompile_assets'
```

The preceding section loads many more specific steps from external files. This is just so that the recipe's code isn't cluttered up unnecessarily. You could put all this in the same file as well. The names of the files should be quite self-explanatory. Note that loading these files only defines which steps are available, it doesn't actually execute them. The execution is specified later.


```ruby
namespace :deploy do
  task :start do
  # we do absolutely nothing here, as we currently aren't
  # using a spinner script or anything of that sort.
  end

  task :restart, :roles => :app, :except => { :no_release => true } do
    run "#{try_sudo} touch #{File.join(current_path,'tmp','restart.txt')}"
  end
  
  # This overwrites the (broken, when using Bundler) deploy:migrate task
    task :migrate do
  end

end
```

This bit makes sure the application server is restarted/reloaded after deployment is complete. It does this by touching `tmp/restart.txt` which triggers Phusion Passenger to restart that specific instance.



```ruby
before "deploy", "retrieve_db_config"
before "deploy:cold", "retrieve_db_config"

before "deploy:create_symlink", :link_config
before "deploy:create_symlink", :link_attachments
before "deploy:create_symlink", :link_db_backups
before "deploy:create_symlink", :chmod_tmp

after "link_config", :migrate_database
after "link_config", :modify_config
after "link_config", "precompile_assets"

before "deploy:restart", :make_tmp

after "deploy", "deploy:cleanup"
```

This is the meat of the recipe: Here we actually call some of the tasks that were loaded from recipe files above. When you run `cap production deploy` you will see each of these steps and each of the step groups listed in order of execution. That can help you decide where to hook something if you want to expand the deployment recipe. Capistrano doesn't make many assumptions here, it's your job to bring things into a sequence that makes sense (e.g. you can't migrate the database if you don't have a valid database configuration file).
