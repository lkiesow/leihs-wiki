# Feature Tests aka Scenarios

* **Write testable application code**
   * Implement features with testing in mind.
   * Don't be afraid to write line of code whose purpose is to make a maintainable and stable test, although it does not make much sense in the application environment (e.g. certain CSS class attributes in favor of `capybara` and/or `selenium` web driver).

* **Don't use personas**:
   * Use factories instead.
   * The DB should be empty upon running a scenario (or with minimal test data, like settings, etc.).
   * Drop steps like `Given I am Mike`. Use `Given I am logged in as an inventory manager`.

* **Gherkin scenarios**
   * Use descriptive and proper name.
   * Keep it short.
   * Write steps sequence describing what a user is doing in terms of UI interaction (translates good to `Capybara` and enables small and reusable steps).
   * Aim only to test what is specified in the name of the scenario => keep scope!
   * There should ideally be only one scenario testing a particular feature. everything else should be defined of terms of scenario prerequisites (`Given ...`).

* **Implementing steps**
   * Use framework which enables modularization (isolation) of feature steps (like e.g. `turnip`).
   * Avoid reusing steps inside another step; use helper methods instead.
   * Keep common steps as small as possible.
   * A common step should have a small implementation and be scenario agnostic (thus reusable in a stable manner).

* **Test data**
   * Avoid randomization! if you need a test with randomized data then make an explicit scenario for that.
   * Each scenario should prepare its own test data.
   * The test data should be minimal => only what is required for the scope of the scenario.
   * Scenarios testing functionality for big data load (or performance) should be separate.

* **Dealing with test results**
   * Test results on Cider-CI are more important than those on your local machine. if you can't get it green on Cider-CI, then delete or disable the test.
