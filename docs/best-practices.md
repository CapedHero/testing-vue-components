# Best Practices

## Prefer `mount` to `shallowMount`

- [https://v1.test-utils.vuejs.org/guides/common-tips.html#shallow-mounting](https://v1.test-utils.vuejs.org/guides/common-tips.html#shallow-mounting)

  > Notice that using `shallowMount` will make the component under testing
  different from the component you run in your application – some of
  its parts won't be rendered! This is why it is not the suggested way of
  testing components unless you face performance issues or need to simplify
  test arrangements.

- [https://github.com/goldbergyoni/javascript-testing-best-practices#-️-33-whenever-possible-test-with-a-realistic-and-fully-rendered-component](https://github.com/goldbergyoni/javascript-testing-best-practices#-%EF%B8%8F-33-whenever-possible-test-with-a-realistic-and-fully-rendered-component)

  > Whenever reasonably sized, test your component from outside like your
  users do, fully render the UI, act on it and assert that the rendered UI
  behaves as expected. Avoid all sort of mocking, partial and shallow
  rendering - this approach might result in untrapped bugs due to lack of
  details and harden the maintenance as the tests mess with the internals.

## Finding the tested element

- Use `data-role` attribute e.g. `<AccountPreview data-role="account-preview">`

    - This means that the above example you would find via:

        ```js
        const accountPreview = wrapper.find('[data-role="account-preview"]')
        ```

- `data-role` is independent of the implementation details like component
  name, HTML tag, CSS classes, or JavaScript ID. Thus, it offers a robust,
  test-specific attribute. As a bonus, `data-role` adds extra clarity regarding
  what is the purpose of a given component or HTML element.

## Basic Test Suite Structure

```js
/*************/
/* VUE SETUP */
/*************/

// Configure Vue for testing.
// Mostly, it's about properly wiring localVue instance.

/***************/
/* TEST VALUES */
/***************/

// Put here ALL of the values used in the tests. This will provide additional 
// information via clear constants names, as well as get rid entirely of magical 
// numbers and similar.

const testCountry = {
  code: 'UK',
  currency: '£',
  currencyCode: 'GBP',
  name: 'United Kingdom',
}
const testEmail = 'test@email.com'
const testId = 123

/***************/
/* MOCKS/STUBS */
/***************/

// Place all but the simplest mocks and stubs in this separate section.
// In result, those test doubles will have a clear interface and will 
// not "pollute" the tests bodies.

function createSentryMock() {
  // ...logic
  return mock
}

class axiosStub {
  // ...logic
}

/***********/
/* HELPERS */
/***********/

// The goal of this section is to extract the tested component
// creation logic, so it has a clear interface and does not 
// "pollute" the tests bodies.
//
// Note: Thus, perhaps, this section should be called CREATORS?

async function createFoo() {
  return await mount('Foo')
}

async function createEmptyFoo() {
  return await mount('Foo', {propsData: {isEmpty: true}})
}

/*********/
/* TESTS */
/*********/

describe('Test group 1', () => {
  test('Test 1', () => {
    // ...logic
  })
})
```

## Basic test structure

- It is recommended to follow `ARRANGE-ACT-ASSERT` pattern.
- [https://automationpanda.com/2020/07/07/arrange-act-assert-a-pattern-for-writing-good-tests/](https://automationpanda.com/2020/07/07/arrange-act-assert-a-pattern-for-writing-good-tests/)

```js
describe('Test group 1', () => {
  test('Test 1', () => {
    // Arrange
    //
    // Here create a wrapper, configure mocks, extract nested values, etc.

    // Pre-Assert
    //
    // This section is quite unorthodox. It can come useful when we want to 
    // ensure that acting on a component really changes something in the 
    // DOM. For instance, if we want to test if a modal appears after clicking
    // a button, it might wise to test here if the modal wasn't already rendered 
    // in the first place.

    // Act
    //
    // Here we act on the component. Usually, this means triggering an action.
    // Note that this section might not be used at all, as there would be 
    // no specific action to test. This happens quite often when testing 
    // the rendering.

    // Assert
    //
    // Here come all of the jest expect statements with which we assert
    // that the component is working as intended.
  })
})
```
