leihs is covered by a suite of automated tests. As outlined in [Contributing](https://github.com/leihs/leihs/wiki/Contributing), if you add new code, you must also cover it with tests.

Running a Cucumber scenario can be done like this:

    bundle exec cucumber features/name_of_feature.feature

The procurement elements of leihs use Rspec with [turnip](https://github.com/jnicklas/turnip). Running such a test works much the same as running a normal Rspec spec file:

    cd engines/procurement
    bundle exec rspec spec/settings.feature

It is expected that most of the leihs test suite will be migrated from Cucumber to Rspec/turnip in future.