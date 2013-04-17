**NOTE:** This guide is not yet complete, but we're working on it and adding sections whenever we encounter something that might interest other developers. If you happen to run into a problem or discover an undocumented workflow during development, please document it here.


# Tests

To run our test suite, you need to make sure the directories for the test result reports exist:

    $ mkdir tmp/html tmp/junit tmp/capybara

Afterwards, it's easy to run the entire test suite:

    $ RAILS_ENV=test bundle exec rake leihs:test

This will first run the Rspec tests (they are more low-level), and if those are successful, the Cucumber tests, examples and scenarios (those are higher-level).

You can also run a single rspec test:


    $ RAILS_ENV=test bundle exec rspec spec/something_spec.rb


Or a single Cucumber scenario or feature file:
    
    $ RAILS_ENV=test bundle exec cucumber -p all features/examples/something.feature

Please run the entire test suite before and after making any changes to the code, so you can detect whether your change introduced any defects into the code, and also so you know whether those defects weren't there already when you got it :)

# Translation

Our translation system is [gettext](http://www.gnu.org/software/gettext/). We chose gettext because simpler, more primitive translation systems usually find out that they are not good enough for the tough world of internationalization, and then go on to reinvent gettext. Since gettext has already done that more than a decade ago, we stick with it. Also, it's easier to get translation files in .po format from professional software translation services.

## Marking strings for translation while developing

In our HTML, ERB and HAML files, just use the usual gettext methods (e.g. `_("foo")`) to mark strings for translation. See Michael Grosser's [fast_gettext](https://github.com/grosser/fast_gettext) for more documentation on gettext, e.g. how to automatically adapt plurals in strings.

In JavaScript and CoffeeScript files, we use [Jed](http://slexaxton.github.com/Jed/) for gettext-like translation.

## Locale files

The locale files are located in the locale directory. To translate leihs to your language, edit the appropriate .po file in the `locale` directory. For example, to translate to Spanish, edit `locale/es/leihs.po`.

.po files can be edited with a wide variety of specialized editing tools. We recommend [poedit](http://www.poedit.net/) because it works the same way on many different operating systems.

### Adding missing translations in existing .po files

Do **not** use the `rake gettext:find` task, as it seems to have a bug at this time that destroys strings found in HAML views.

Instead, install gettext on your system to get the msgfmt and msgmerge utilities. Add any new strings that you've added to leihs to the locale/leihs.pot file. Then Merge with the existing target language that you want to translate to. For example:

    msgmerge de_CH/leihs.po leihs.pot > de_CH/leihs.po.new

Now examine leihs.po.new and rename it to leihs.po if you're happy with the merged result. Then go on to open the .po file for your language in a text editor or your favorite .po file editor. Newly found strings might be marked as fuzzy. Translate the strings, unfuzzy them and save the file.

If you have your own fork of leihs, push your contribution to your fork and send us a pull request. You don't need to worry about any further steps, we'll take care of the rest. 

#### Translating JavaScript

JavaScript is a special case, as we needed to alias the gettext `_()` method in our templates. Instead, if you introduce a new string to any JavaScript template, please call the `_jed()` function on it. So anything you would use `_()` for in Ruby needs to be done with `_jed()` in JavaScript. Please check out the [Jed](http://slexaxton.github.com/Jed/) documentation to find out more about what options you have in translating JavaScript strings.

Now consult _Adding missing translations in existing .po files_ to find out how to add translations to the system. This is generally enough to cover the JavaScript strings.

If you want to be extra nice and have Node.js installed on your system, you can now immediately convert the .po files to .json files for Jed to use:

    $ bundle exec rake app:i18n:po2json

### Translating to a new language

If your language does not exist yet, create a new directory according to the [ISO-639 standard language code](http://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) for your language, optionally followed by an underscore and a region code for your region. For example, to translate to Romanian, create the directory `locale/ro`.

Next, copy the translation template file locale/leihs.pot into the new directory using a .po extension, not .pot! For example:

    $ cp locale/leihs.pot locale/ro/leihs.po

Now continue as described under _Adding missing translations in existing .po files_.


