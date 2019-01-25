## Existing ones

leihs comes with a set of predefined languages as defined in [this peace of code](https://github.com/leihs/leihs_legacy/blob/master/features/support/leihs_factory.rb#L247-L250).

```
leihs=# select * from languages;
                  id                  |     name     | locale_name | default | active
--------------------------------------+--------------+-------------+---------+--------
 36a364e1-0aea-470e-98ee-9591d45d5173 | English (UK) | en-GB       | t       | t
 916325ef-0786-4561-a6f6-8bb250dba938 | English (US) | en-US       | f       | t
 9c9ead79-9bbf-4652-a52a-2cc77c661b3e | Deutsch      | de-CH       | f       | t
 eba33ac2-09ae-4a50-800a-93d6e4ff87dc | Züritüütsch  | gsw-CH      | f       | t
```

## Desire for a new one

A developer can extend leihs with a new language following [this guide](https://github.com/leihs/leihs/wiki/Developer-Guide#translating-to-a-new-language).

## Configuration

Each individual leihs instance can activate/deactive the existing languages as well as set its default one by configuring its deployment inventory accordingly:
```yaml
active_locales:
  de-CH: true
  en-GB: true
  es: true
  gsw-CH: true
  en-US: false

default_locale: "de-CH"
```