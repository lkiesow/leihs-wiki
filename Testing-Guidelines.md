# Feature Tests aka Scenarios (WIP)

* **don't use personas**:
 * use factories instead
 * the DB should be empty upon running a scenario (or with minimal test data, like settings, etc.)
 * drop steps like `Given I am Mike`. Use `Given I am logged in as an inventory manager`.

* **Gherkin scenarios**
 * use descriptive and proper name
 * keep it short
 * write steps sequence describing what a user is doing in terms of UI interaction (translates good to `Capybara` and enables small and reusable steps)
 * aim only to test what is specified in the name of the scenario => keep scope!
 * there should ideally be only one scenario testing a particular feature. everything else should be defined of terms of scenario prerequisites (`Given ...`)

* **implementing steps**
 * use framework which enables modularization (isolation) of feature steps (like e.g. `turnip`)
 * avoid reusing steps inside another step; use helper methods instead
 * keep common steps as small as possible
 * a common step should have a small implementation and be scenario agnostic (thus reusable in a stable manner)

* **test data**
 * avoid randomization! if you need a test with randomized data then make an explicit scenario for that.
 * each scenario should prepare its own test data
 * the test data should be minimal => only what is required for the scope of the scenario
 * scenarios testing functionality for big data load (or performance) should be separate