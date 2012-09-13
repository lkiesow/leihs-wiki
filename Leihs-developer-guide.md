**NOTE:** This guide is not yet complete, but we're working on it and adding sections whenever we encounter something that might interest other developers. If you happen to run into a problem or discover an undocumented workflow during development, please document it here.


# Translation

Our translation system is [gettext](http://www.gnu.org/software/gettext/). We chose gettext because simpler, more primitive translation systems usually find out that they are not good enough for the tough world of internationalization, and then go on to reinvent gettext. Since gettext has already done that more than a decade ago, we stick with it. Also, it's easier to get translation files in .po format from professional software translation services.

## Marking strings for translation while developing

In our HTML, ERB and HAML files, just use the usual gettext methods (e.g. `_("foo")`) to mark strings for translation. See Michael Grosser's [fast_gettext](https://github.com/grosser/fast_gettext) for more documentation on gettext, e.g. how to automatically adapt plurals in strings.

In JavaScript and CoffeeScript files, we use [Jed](http://slexaxton.github.com/Jed/) for gettext-like translation. If you add a new string to a JavaScript or CoffeeScript file, you must also add it to app/views/javascript_strings.html.erb as a Ruby string. Sorry about that! The reason for this is that there are currently no string extraction tools that can reliably extract translation strings from JavaScript files. Putting them in a Ruby file makes sure that the gettext:find rake task finds them and that they are added to the .po files for all languages.

## Locale files

The locale files are located in the locale directory. To translate leihs to your language, edit the appropriate .po file in the `locale` directory. For example, to translate to Spanish, edit `locale/es/leihs.po`.

.po files can be edited with a wide variety of specialized editing tools. We recommend [poedit](http://www.poedit.net/) because it works the same way on many different operating systems.

### Adding missing translations in existing .po files

Run the gettext:find task to find new untranslated strings in all the views:

    $ bundle exec rake gettext:find

Ruby-gettext will search through the templates and merge what it finds with the existing translations. The output should look something like this:

    locale/leihs.pot .................................................. done.
    locale/es/leihs.po .................................................. done.
    locale/de_CH/leihs.po .................................................. done.
    locale/en_GB/leihs.po .................................................. done.
    locale/gsw_CH/leihs.po .................................................. done.
    locale/en_US/leihs.po ................................................... done.

Then go on to open the .po file for your language in a text editor or your favorite .po file editor. Newly found strings might be marked as fuzzy. Translate the strings, unfuzzy them and save the file.

If you have your own fork of leihs, push your contribution to your fork and send us a pull request. You don't need to worry about any further steps, we'll take care of the rest. If you want to be extra-nice, you can pack your new translations into a .mo file using the gettext:pack task:

    $ bundle exec rake gettext:pack
    
This goes through each .po file and creates a matching machine-readable and compressed .mo file in the `locale/nn_NN/LC_MESSAGES` directory.

### Translating to a new language

If your language does not exist yet, create a new directory according to the [ISO-639 standard language code](http://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) for your language, optionally followed by an underscore and a region code for your region. For example, to translate to Romanian, create the directory `locale/ro`.

Next, copy the translation template file locale/leihs.pot into the new directory using a .po extension, not .pot! For example:

    $ cp locale/leihs.pot locale/ro/leihs.po

Now continue as described under _Adding missing translations in existing .po files_.