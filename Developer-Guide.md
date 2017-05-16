Use this guide if you want to setup developer environment on your local machine. For installing leihs on a server use following [instructions](https://github.com/leihs/leihs/wiki/Deployment).

# Code setup

```
git clone --recursive https://github.com/leihs/leihs.git
cd leihs
git checkout origin/v4
git submodule update --init --recursive
```

# Rbenv and bundler setup

1. `cd legacy`
1. Install [rbenv](https://github.com/rbenv/rbenv) and follow the instruction on the home page to set the correct Ruby version (see `deploy` submodule).
2. `gem install bundler`
3. `bundle install`

# Database setup

1. Install [PostgreSQL](https://www.postgresql.org/).
2. Create and/or change `legacy/config/database.yml` according to your needs. You might take a look at the example file: `legacy/config/database_pg.yml`.

# Data setup

1. `bundle exec rake db:create`
2. Load our personas test data (and structure) into the development database:
`bundle exec rake db:pg:structure_and_data:restore FILE=features/personas/personas.pgbin`

# Running Rails server

1. `bundle exec rails server`
2. You can login for example as an admin user "gino". All users have password "password".

# Running the tests

Create your test database, it will be populated later with database dumps representing a known state of the system:

    RAILS_ENV=test bundle exec rake db:create

Running a manage/borrow feature or scenario:

    bundle exec cucumber --strict features/borrow/calendar.feature:10

Running an admin feature or scenario:

    bundle exec rspec -r ./engines/leihs_admin/spec/load.rb engines/leihs_admin/spec/features/buildings.feature:13

Running a procurement feature or scenario:

    bundle exec rspec -r ./engines/procurement/spec/load.rb engines/procurement/spec/features/categories.feature:21

# Static code checks

Running *rubocop* checks for borrow/manage:

    bundle exec rubocop

Running *rubocop* checks for admin:

    bundle exec rubocop engines/leihs_admin/ -c engines/leihs_admin/.rubocop.yml

Running *rubocop* checks for procurement:

    bundle exec rubocop engines/procurement/ -c engines/procurement/.rubocop.yml

Besides you can run:
- `flay app/ engines/leihs_admin/app engines/procurement/app`
- `flog app/ engines/leihs_admin/app engines/procurement/app --continue`

Install them with `gem install flay` and `gem install flog`.

# Translation

Our translation system is [gettext](http://www.gnu.org/software/gettext/).

## Marking strings for translation while developing

In our HTML, ERB and HAML files, just use the usual gettext methods (e.g. `_("foo")`) to mark strings for translation. See Michael Grosser's [fast_gettext](https://github.com/grosser/fast_gettext) for more documentation on gettext, e.g. how to automatically adapt plurals in strings.

In JavaScript and CoffeeScript files, we use [Jed](http://slexaxton.github.com/Jed/) for gettext-like translation.

## Locale files

The locale files are located in the locale directory. To translate leihs to your language, edit the appropriate .po file in the `locale` directory. For example, to translate to Spanish, edit `locale/es/leihs.po`.

.po files can be edited with a wide variety of specialized editing tools. We recommend [poedit](http://www.poedit.net/) because it works the same way on many different operating systems.

## Adding missing translations in existing .po files

Do **not** use the `rake gettext:find` task, as it seems to have a bug at this time that destroys strings found in HAML views.

Instead, install gettext on your system to get the msgfmt and msgmerge utilities. Add any new strings that you've added to leihs to the locale/leihs.pot file. Then Merge with the existing target language that you want to translate to. For example:

    msgmerge de_CH/leihs.po leihs.pot > de_CH/leihs.po.new

Now examine leihs.po.new and rename it to leihs.po if you're happy with the merged result. Then go on to open the .po file for your language in a text editor or your favorite .po file editor. Newly found strings might be marked as fuzzy. Translate the strings, unfuzzy them and save the file.

If you have your own fork of leihs, push your contribution to your fork and send us a pull request. You don't need to worry about any further steps, we'll take care of the rest.

## Translating JavaScript

JavaScript is a special case, as we needed to alias the gettext `_()` method in our templates. Instead, if you introduce a new string to any JavaScript template, please call the `_jed()` function on it. So anything you would use `_()` for in Ruby needs to be done with `_jed()` in JavaScript. Please check out the [Jed](http://slexaxton.github.com/Jed/) documentation to find out more about what options you have in translating JavaScript strings.

Now consult _Adding missing translations in existing .po files_ to find out how to add translations to the system. This is generally enough to cover the JavaScript strings.

If you want to be extra nice and have Node.js installed on your system, you can now immediately convert the .po files to .json files for Jed to use:

    $ bundle exec rake app:i18n:po2json

## Translating to a new language

If your language does not exist yet, create a new directory according to the [ISO-639 standard language code](http://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) for your language, optionally followed by an underscore and a region code for your region. For example, to translate to Romanian, create the directory `locale/ro`.

Next, copy the translation template file locale/leihs.pot into the new directory using a .po extension, not .pot! For example:

    $ cp locale/leihs.pot locale/ro/leihs.po

Now continue as described under _Adding missing translations in existing .po files_.