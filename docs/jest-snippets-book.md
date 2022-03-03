# Jest Snippets Book

## Registering 3rd party components, plugins, mixins, directives, etc.

```js
import { createLocalVue, mount } from '@vue/test-utils'

// https://v1.test-utils.vuejs.org/api/createlocalvue.html#createlocalvue
const localVue = createLocalVue()
localVue.use('*3RD PARTY MODULE*')
s
test('test', async () => {
  const wrapper = await mount(Foo, { localVue })
})
```

### Special case: Vuetify

```js
// Globa Vuetify registration should be called once in test/setup.js
Vue.use(Vuetify) 

const localVue = createLocalVue()
localVue.use(Vuetify)
```

### Special case: Error in v-on handler: "TypeError: Cannot read properties of undefined (reading '_transitionClasses')”

```js
const wrapper = await mount('Foo', { sync: false })
```

!!! note
     
    Is this stil necessary after using the latest `vue-test-utils`?

## Injecting props and data to tested component

```js
async function createComponentFoo() {
  return await mount(
    Foo,
    {
      propsData: {
        bar: 'bar',
      },
      data: () => ({
        baz: 'baz',
      }),
    },
  )
}
```

## Stubbing child components

- It’s especially useful for stubbing Font Awesome or specific Vuetify
  components.

```js
// https://v1.test-utils.vuejs.org/api/options.html#stubs
async function createComponentFoo() {
  const wrapper = await mount(
    Foo,
    {
      stubs: {
        fa: true,
        BaseButton: true,
      },
    },
  )

  return wrapper
}
```

## Finding DOM elements

```js
const wrapper = await mount('Foo')
const oneEl = wrapper.find('*VALID-SELECTOR*')
const allEls = wrapper.findAll('*VALID-SELECTOR*')
const firstElFromAllEls = allEls.at(0)
```

## Settings form values

```js
const inputFoo = wrapper.find('[data-role="form-input-foo"]')
inputFoo.setValue('foo')
```

## Trigger native DOM event

```js
wrapper.trigger('*DOM-EVENT*')
wrapper.trigger('click')
```

## Trigger custom events

```js
wrapper.vm.$emit('*CUSTOM-EVENT*')
wrapper.vm.$emit('add-account')
```

## Waiting for DOM to re-rerender

If the tested component does not show what you expect, it might be possible
that you must explicitly wait for the DOM to re-rerender.

This is especially the case after triggering clicks, popping
modals/alerts/errors, or with larger component when it might take a couple
rendering cycles for the DOM to fully render.

In such cases, first await the method that you mutated the state with, e.g.:

```js
await wrapper.trigger('click')
```

!!! attention

    `mount` is also awaitable! As a result, always call `await mount(Foo)`.

If this doesn't work, try to explicitly wait until Vue.js has finished updating
the DOM after a data change:

```js
await Vue.nextTick()
```

Sometimes it requires more than one rendering cycle to fully render the state
you want to test. In such cases, call:

```js
for (let i = 0; i < 3; i++) {
  await Vue.nextTick()
}
// In my personal experience, it never needs more than 3 rendering cycles.
```

## Most useful mocks

### Mocking $router (or any other dollar attribute like $env)

```js
async function createComponentFoo() {
  const routerPushMock = jest.fn()
  const testRouter = { push: routerPushMock }

  const wrapper = await mount(
    Foo,
    {
      localVue,
      mocks: { $router: testRouter },
    },
  )

  return { wrapper, routerPushMock }
}
```

### Mocking Vuex store

```js
async function createComponentFoo() {
  const setBarMock = jest.fn()
  const store = new Vuex.Store({
    actions: {
      'module1/setBar': setBarMock,
    },
    getters: {
      'module1/getBaz': () => 'baz',
    },
  })

  const wrapper = await mount(
    Foo,
    {
      localVue,
      mocks: { $router: testRouter },
    },
  )

  return { wrapper, setBarMock }
}
```

### Mocking `fetch`

```js
describe('Test group 1', () => {
  beforeEach(() => {
    fetchMock.resetMocks()
  })

  test('Test 1', () => {
    // Logic that is using fetch

    expect(fetch.mock.calls[0][0]).toEqual('expectedUrl')
    expect(fetch.mock.calls[0][1].method).toEqual('GET')
    expect(fetch.mock.calls[0][1].body).toEqual(JSON.stringify(expectedBody))
  })
})
```

### Mock “error” response from `axios`

```js
function createAxiosGetMockThrowingError400() {
  const axiosGetMockError = new Error('Test Axios GET 400 Error')
  axiosGetMockError.statusCode = 400

  return jest.fn().mockImplementation(() => Promise.reject(axiosGetMockError))
}
```

### Spying on method calls

```js
const methodSpy = jest.spyOn(Foo.methods, 'methodToSpyOn')
```

## Testing with `expect`

### Most useful `expect` validators

```js
// https://jestjs.io/docs/expect

expect(string).toContain(text)

expect(string).toMatch(textPattern)

expect(clientId).toEqual(123)

expect(client).toEqual({foo: 'foo'})

expect(isShown).toBe(true)

expect(array).toHaveLength(3)

expect(spy).toHaveBeenCalled()

expect(jestMock.mock.calls[callNum][callArgIndex]).toEqual({expectedPayload})

// Negation
expect(clientId).not.toEqual(456)
```

### Common `expect` formulas

```js
// Check if el exists
expect(wrapper.exists()).toBe(true)

// Check if el text matches regex pattern
expect(wrapper.text()).toMatch(textPattern)

// Check if el is disabled
expect(!!wrapper.attributes('disabled')).toBe(true)

// Check if el emitted events
const targetEvents = wrapper.emitted()['eventName']
const targetEvent = targetEvents.length ? targetEvents[0] : null
expect(!!event).toBe(true)

const actualEventPayload = event[0]
const expectedEventPayload = { foo: 'foo' }
expect(actualEventPayload).toEqual(expectedEventPayload)
```

## Test parametrization

[https://jestjs.io/docs/api#testeachtablename-fn-timeout](https://jestjs.io/docs/api#testeachtablename-fn-timeout)

```js
test.each`
  a    | b    | expected
  ${1} | ${1} | ${2}
  ${1} | ${2} | ${3}
  ${2} | ${1} | ${3}
`('returns $expected when $a is added $b', ({ a, b, expected }) => {
  expect(a + b).toBe(expected);
});
```

```js
test.each([
  [1, 1, 2],
  [1, 2, 3],
  [2, 1, 3],
])('.add(%i, %i)', (a, b, expected) => {
  expect(a + b).toBe(expected);
});
```

## Master Template

```js
import VTooltip from 'v-tooltip'
import Vuetify from 'vuetify'
import Vuex from 'vuex'
import { createLocalVue, mount } from '@vue/test-utils'

/*************/
/* VUE SETUP */
/*************/

const localVue = createLocalVue()
localVue.use(Vuetify)
localVue.use(Vuex)
localVue.use(VTooltip)

/***************/
/* TEST VALUES */
/***************/

/*********/
/* MOCKS */
/*********/

/************/
/* WRAPPERS */
/************/

async function createFoo() {
  return await mount(
    Foo,
    {
      localVue,
      propsData: {
        bar: 'bar',
      },
      data: () => ({
        baz: 'baz',
      }),
      stubs: { fa: true },
    },
  )
}

/*********/
/* TESTS */
/*********/

describe('Test group 1', () => {
  test('Test 1', async () => {
    // Arrange
    const wrapper = await createFoo()
    
    // Pre-Assert

    // Act

    // Assert
  })
})
```
