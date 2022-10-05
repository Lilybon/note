# Jest Common Usage

Since JSTF (our frontend policy team) adds **SonarQube** for all FE projects recently. We have to get started with unit test for better code coverage in all projects. Any PR doesn't reach a goal of 60% coverage can't be merged.<br/>

To reduce the pain and writing unit test in an efficient way, here are some common cases for `jest` and `vue-test-util`。

### Jest

Q: How to mock **window**?<br/>
A:

```javascript
// TODO
```

Q: If there are multiple test cases in a simular pattern, how do you test all of them?<br/>
A:

```javascript
// TODO
```

Q: How to end up all promises in jest?<br/>
A:

```javascript
import flushPromises from 'flush-promises'

it('should end all promises', async () => {
  anAsyncFunction()
  await flushPromises()
  // ...
})
```

Q: How to mock third party **ES Module**?<br/>
A:

```javascript
jest.mock('', () => ({
  // TODO
}))

jest.mock('', () => ({
  // TODO
}))
```

### Vue Test Util

Q: How to mock **router**？<br/>
A:

```javascript
// If your code has some router methods such as pop and replace, mock it with jest.fn is fine.
const mockRouter = {
  push: jest.fn(),
}

const localVue = createLocalVue()

const wrapper = shallowMount(Foo, {
  localVue,
  global: {
    mocks: {
      $router: mockRouter,
    },
  },
})
```

Q: How to mock **route**？<br/>
A:

```javascript
const mockRoute = {
  path: '/foo',
}

const localVue = createLocalVue()

const wrapper = shallowMount(Foo, {
  localVue,
  global: {
    mocks: {
      $route: mockRoute,
    },
  },
})
```

Q: How to mock **next** function in **beforeRouteLeave**?<br/>
A:

```javascript
it('should trigger next function in some condition', () => {
  // ...
  const next = jest.fn()
  wrapper.beforeRouteLeave.call(wrapper.vm, to, from, next)
  expect(next).toHaveBeenCalledTimes(1)
})
```

Q: How to get/set **data** in **wrapper**?<br/>
A:

```javascript
it('should set hello to world', async () => {
  // set
  wrapper.setData({
    hello: 'world',
  })

  await wrapper.vm.$nextTick()

  // get
  expect(wrapper.vm.hello).toBe('world')
})
```

Q: How to mock `vue-i18n`?<br/>
A:

```javascript
const localVue = createLocalVue()

jest.mock('', () => ({
  // TODO
}))

const wrapper = shallowMount(Foo, {
  localVue,
  global: {
    mocks: {
      $t: jest.fn((key) => key),
    },
  },
})
```

Q: How to test a **dialog** called by **plugin method**?<br/>
A: Because it's general to see that a dialog called by plugin method is a **Vue** instance seperated with our **App** instance, to access the instance, there is a hacky method. Take **bootstrap-vue** as example, we should realize what **DOM structure** the dialog is by reading the third party **repo** source code, get `__vue__` property in the dialog **DOM**, and then use `createWrapper` method to generate a new **wrapper**。

```javascript
const localVue = createLocalVue()
localVue.use(BootstrapVue)

it('should show bootstrap modal and interact with it', async () => {
  // ...

  // create modal wrapper in a hacky way
  // bootstrap vue modal spec: https://github.com/bootstrap-vue/bootstrap-vue/blob/dev/src/components/modal/modal.spec.js
  const modalElement = document.querySelector('div.modal')
  const modal = createWrapper(modalElement.__vue__)
  expect(modal.exists()).toBeTruthy()

  const button = modal.find('button.btn-primary')
  await button.trigger('click')

  // ...
})
```

###### tags: `Work` `JavaScript` `Jest` `Vue` `Vue2` `Unit Test`
