# Getting Started

## How to create a new test suite?

- Write one minimal test that successfully renders the tested component.

    - The test should not output any errors or warnings.

    - The test should have a minimal setup and minimal input. A good
      verification is that you cannot remove any line because the test will
      immediately fail or start outputting warnings or errors.

- Once you have such a minimal test, copy it and adjust it for specific
  test cases.

## What to test?

- **Rendering**

    - Test whether key headers, texts, errors, etc., are rendered by the
      component.

    - Test every logical branch, including each `v-if`, `v-show`, and `v-for`
      (the latter with an empty and at least two-elements collection).

        - Jest coverage report will be your best friend here. It clearly
          shows which parts of the component were not rendered by the tests,
          which will help you find the next test case.

- **Interaction**

    - Test clicks, swipes, or any other user interaction with the component.

- **Events Handling**

    - Test if the component properly handles the events emitted by its
      children components.

### What about testing logic?

Logic should be tested as part of above-mentioned groups of tests, e.g.:

- If the logic results in changes to the rendered component, then test whether
  specific outputs were rendered.

- If the logic results in side effects, then stub or mock the called functions
  and test whether the expected calls were made. Usually, it is a part of the
  “Interaction” tests. An example would be whether a proper API call was made
  after the form was submitted.

Still, sometimes you will just have to be creative. In such cases, try to
extend the test suite in a readable, logical manner. Consider this example.
You call some important business logic with side effects during component
mounting. This could warrant a couple of tests with mocks in their own
section of “Mounting”.

Moreover, sometimes it is worth extracting the logic to a separate JS module in
order to test it independently of any component rendering. This is especially
applicable when the logic is reusable. Similarly, it could be a good idea to
extract the logic when there are numerous simple cases that are straightforward
to test on their own, while testing in a fully-rendered component would be
wasteful and less readable.
