# Vue unit testing cheat sheet
:microscope: My cheat sheet for testing vue components with jest and vue test utils

Remark: In the time of making this cheat sheet, I had no clue about (unit) testing, so these examples might not be in terms of good/best practices. 

:pray: Many examples in here are taken from the repos, videos and tutorials from [MikaelEdebro](https://github.com/MikaelEdebro), [Edd Yerburgh](https://github.com/eddyerburgh), [Alex Jover](https://github.com/alexjoverm) and others.

Useful links: 
* [jest documentation](https://jestjs.io/docs/en/getting-started)
* [vue-test-utils](https://vue-test-utils.vuejs.org/)
* [globals](https://jestjs.io/docs/en/api)
* [expect](https://jestjs.io/docs/en/expect)
* [jest-serializer-vue-tjw](https://github.com/tjw-lint/jest-serializer-vue-tjw)
* https://github.com/sapegin/jest-cheat-sheet

## A few words before
Where is the right balance between what to test and what not to test? We can consider writinh unit tests in cases like:
* when the logic behind the method is complex enough that you feel you need to test extensively to verify that it works.
* whenever it takes less time to write a unit test to verify that code works than to start up the system, log in, recreate your scenario, etc.
* when there are possible edge cases of unusually complex code that you think will probably have errors
* when a particulary complex function is receiving multiple arguments, it is a good idea to feed that function with null, undefined and unexpected data (e.g. methodNullTest, methodInvalidValueTest, methodValidValueTest)
* when there are cases that require complex steps for reproduction and can be easily forgotten

Try to keep test methods short and sweet and add them to the build.

## Setuping jest 
Follow these steps for the most basic jest setup:
1. ```npm i --save-dev jest```
2. add ```"unit": "jest"``` to your package.json file

   ```json
   {
     "scripts": {
       "unit": "jest",
     },
   } 
   ```
3. create ```Component.spec.js``` file in the component folder
4. add ```jest: true``` to your ```.eslintrc.js``` file (so that eslint knows the jest keywords like describe, expect etc.)

   ```javascript
   {
     env: {
       browser: true,
       jest: true
     },
   }
   ```
5. Write tests in the ```Component.spec.js``` file and run it with ```npm run unit```

### Frequent terminology
* Shallow Rendering - a technique that assures your component is rendering without children. This is useful for:
  * Testing only the component you want to test (that's what Unit Test stands for)
  * Avoid side effects that children components can have, such as making HTTP calls, calling store actions...

### Sanity test
```javascript
describe('Component.vue', () => {
  test('sanity test', () => {
    expect(true).toBe(true)
  })
})
```
### Access Vue component
```javascript
// component access
import { mount } from '@vue/test-utils'
import Modal from '../Modal.vue'
const wrapper = mount(Modal)

wrapper.vm // access to the component instance
wrapper.element // access to the component DOM node
```

### Test Vue components
```javascript
// Import Vue and the component being tested
import Vue from 'vue'
import MyComponent from 'path/to/MyComponent.vue'

describe('MyComponent', () => {
  // Inspect the raw component options
  it('has a created hook', () => {
    expect(typeof MyComponent.created).toBe('function')
  })

  // access the component data 
  it('check init state of "message" from data', () => {
    const vm = new Vue(MyComponent).$mount()
    expect(vm.message).toBe('bla')
  })
  
  // usage of helper functions 
  it('hides the body initially', () => {
    h.domHasNot('body')
  })
  it('shows body when clicking title', () => {
    h.click('title')
    h.domHas('.body')
  })
  it('renders the correct title and subtitle after clicking button', () => {
    h.click('.my-button')
    h.see('My title')
    h.see('My subtitle')
  })
  it('adds expanded class to expanded post', () => {
    h.click('.post')
    expect(wrapper.classes()).toContain('expanded')
  })
  it('show comments when expanded', () => {
    h.click('.post')
    h.domHas('.comment')
  })
})
```

## Test default and passed props
```javascript
// we'll create a helper factory function to create a message component, give some properties
const createCmp = propsData => mount(KaModal, { propsData });

it("has a message property", () => {
  cmp = createCmp({ message: "hey" });
  expect(cmp.props().message).toBe("hey");
});

it("has no cat property", () => {
  cmp = createCmp({ cat: "hey" });
  expect(cmp.props().cat).toBeUndefined();
});

// test default value of author prop
it("Paco is the default author", () => {
  cmp = createCmp({ message: "hey" });
  expect(cmp.props().author).toBe("Paco");
});

// slightly different approach
// pass props to child with 'propsData'
it('renders correctly with different props', () => {
  const Constructor = Vue.extend(MyComponent)
  const vm = new Constructor({ propsData: {msg: 'Hello'} }).$mount()
  expect(vm.$el.textContent).toBe('Hello')
})

// prop type
it("hasClose is bool type", () => {
    const message = createCmp({title: "hey"});
    expect(message.vm.$options.props.hasClose.type).toBe(Boolean);
})

// prop required
it("hasClose is not required", () => {
    const message = createCmp({title: "hey"});
    expect(message.vm.$options.props.hasClose.required).toBeFalsy();
})

// custom events
it("Calls handleMessageClick when @message-click happens", () => {
  const stub = jest.fn();
  cmp.setMethods({ handleMessageClick: stub });
  const el = cmp.find(Message).vm.$emit("message-clicked", "cat");
  expect(stub).toBeCalledWith("cat");
});

```

### Test visibility of component with passed prop
```javascript
test('does not render when not passed visible prop', () => {
  const wrapper = mount(Modal)
  expect(wrapper.isEmpty()).toBe(true)
})
test('render when visibility prop is true', () => {
  const wrapper = mount(Modal, {
    propsData: {
      visible: true
    }
  })
  expect(wrapper.isEmpty()).toBe(false)
})
test('call close() method when X is clicked', () => {
  const close = jest.fn()
  const wrapper = mount(Modal, {
    propsData: {
      visible: true,
      close
    }
  })
  wrapper.find('button').trigger('click')
  expect(close).toHaveBeenCalled()
})
```

### Test computed properties
```javascript
it("returns the string in normal order", () => {
  cmp.setData({ inputValue: "Yoo" });
  expect(cmp.vm.reversedInput).toBe("Yoo");
});
```

### Vuex actions 
Before testing anything from the vuex store, we need to "mock" (make dummy / hardcode) the store values that we want to test. In the case beneath, we simulated the result of the getComments async action to give us 2 comments.
```javascript
import Vuex from 'vuex'
import { shallow, createLocalVue } from '@vue/test-utils'
import BlogComments from '@/components/blog/BlogComments'
import TestHelpers from 'test/test-helpers'
import Loader from '@/components/Loader'
import flushPromises from 'flush-promises'

const localVue = createLocalVue()
localVue.use(Vuex)

describe('BlogComments', () => {
  let wrapper
  let store
  // eslint-disable-next-line
  let h
  let actions
  beforeEach(() => {
    actions = {
      getComments: jest.fn(() => {
        return new Promise(resolve => {
          process.nextTick(() => {
            resolve([{ title: 'title 1' }, { title: 'title 2' }])
          })
        })
      })
    }
    store = new Vuex.Store({
      modules: {
        blog: {
          namespaced: true,
          actions
        }
      }
    })
    wrapper = shallow(BlogComments, {
      localVue,
      store,
      propsData: {
        id: 1
      },
      stubs: {
        Loader
      },
      mocks: {
        $texts: {
          noComments: 'No comments'
        }
      }
    })
    h = new TestHelpers(wrapper, expect)
  })

  it('renders without errors', () => {
    expect(wrapper.isVueInstance()).toBeTruthy()
  })

  it('calls action to get comments on mount', () => {
    expect(actions.getComments).toHaveBeenCalled()
  })

  it('shows loader initially, and hides it when comments have been loaded', async () => {
    h.domHas(Loader)
    await flushPromises()
    h.domHasNot(Loader)
  })

  it('has list of comments', async () => {
    await flushPromises()
    const comments = wrapper.findAll('.comment')
    expect(comments.length).toBe(2)
  })

  it('shows message if there are no comments', async () => {
    await flushPromises()
    wrapper.setData({
      comments: []
    })
    h.domHas('.no-comments')
    h.see('No comments')
  })
})
```
### Vuex mutations and getters
Don't forget to correctly export mutations and getters so they can be accessible in the tests
```javascript
import { getters, mutations } from '@/store/modules/blog'

describe('blog store module', () => {
  let state
  beforeEach(() => {
    state = {
      blogPosts: []
    }
  })
  describe('getters', () => {
    it('hasBlogPosts logic works', () => {
      expect(getters.hasBlogPosts(state)).toBe(false)
      state.blogPosts = [{}, {}]
      expect(getters.hasBlogPosts(state)).toBe(true)
    })
    it('numberOfPosts returns correct count', () => {
      expect(getters.numberOfPosts(state)).toBe(0)
      state.blogPosts = [{}, {}]
      expect(getters.numberOfPosts(state)).toBe(2)
    })
  })

  describe('mutations', () => {
    it('adds blog posts correctly', () => {
      mutations.saveBlogPosts(state, [{ title: 'New post' }])
      expect(state.blogPosts).toEqual([{ title: 'New post' }])
    })
  })
})
```

### Snapshot test
From the official jest documentation:

*Snapshot tests are a very useful tool whenever you want to make sure your UI does not change unexpectedly.
A typical snapshot test case for a mobile app renders a UI component, takes a screenshot, then compares it to a reference image stored alongside the test. The test will fail if the two images do not match: either the change is unexpected, or the screenshot needs to be updated to the new version of the UI component.*

```javascript
test('snapshot test', () => {
  const wrapper = mount(Modal, {
    propsData: {
      visible: true
    }
  })
  // wrapper.html() -> string of the html of our mounted component
  expect(wrapper.html()).toMatchSnapshot()
})
```

### Testing helper functions 
```javascript
import { sort } from './helpers'

describe('Helper functions', () => {
  test('it sorts the array', () => {
    const arr = [3,1,5,4,2]
    const res = sort(arr, 'asc')
    expect(res).toEqual(arr.sort((a,b) => a-b))
  })
})
```

### Useful [helpers](https://github.com/MikaelEdebro/ezy-dev-session-jest)
```javascript
class TestHelpers {
  constructor(wrapper, expect) {
    this.wrapper = wrapper
    this.expect = expect
  }

  see(text, selector) {
    let wrap = selector ? this.wrapper.find(selector) : this.wrapper
    this.expect(wrap.html()).toContain(text)
  }
  doNotSee(text) {
    this.expect(this.wrapper.html()).not.toContain(text)
  }
  type(text, input) {
    let node = this.find(input)
    node.element.value = text
    node.trigger('input')
  }
  click(selector) {
    this.wrapper.find(selector).trigger('click')
  }
  inputValueIs(text, selector) {
    this.expect(this.find(selector).element.value).toBe(text)
  }
  inputValueIsNot(text, selector) {
    this.expect(this.find(selector).element.value).not.toBe(text)
  }
  domHas(selector) {
    this.expect(this.wrapper.contains(selector)).toBe(true)
  }
  domHasNot(selector) {
    this.expect(this.wrapper.contains(selector)).toBe(false)
  }
  domHasLength(selector, length) {
    this.expect(this.wrapper.findAll(selector).length).toBe(length)
  }
  isVisible(selector) {
    this.expect(this.find(selector).element.style.display).not.toEqual('none')
  }
  isHidden(selector) {
    this.expect(this.find(selector).element.style.display).toEqual('none')
  }
  find(selector) {
    return this.wrapper.find(selector)
  }
  hasAttribute(selector, attribute) {
    return this.expect(this.find(selector).attributes()[attribute]).toBeTruthy()
  }
}

export default TestHelpers
```
```javascript
// how to use helpers
import TestHelpers from 'test/test-helpers'
let h = new TestHelpers(wrapper, expect)
h.domHas('.loader')
```
