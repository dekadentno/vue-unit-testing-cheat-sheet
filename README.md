# jest-cheat-sheet
:microscope: My cheat sheet for testing vue components with jest

Remark: I have no clue about (unit) testing, so these examples might not be in terms of good/best practices. 

Useful links: 
* [jest documentation](https://jestjs.io/docs/en/getting-started)
* [globals](https://jestjs.io/docs/en/api)
* [expect](https://jestjs.io/docs/en/expect)

## Setuping jest 
Follow these steps for the most basic jest setup:
1. ```npm i --save-dev jest```
2. add ```"unit": "jest"``` to your package.json file
3. create ```Component.spec.js``` file in the component folder
4. add ```jest: true``` to your ```.eslintrc.js``` file (so that eslint knows the jest keywords like describe, expect etc.)
5. Write tests in the ```Component.spec.js``` file and run it with ```npm run unit```

### Sanity test
```javascript
describe('Component.vue', () => {
  test('sanity test', () => {
    expect(true).toBe(true)
  })
})
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
  
  // pass props to child with 'propsData'
  it('renders correctly with different props', () => {
    const Constructor = Vue.extend(MyComponent)
    const vm = new Constructor({ propsData: {msg: 'Hello'} }).$mount()
    expect(vm.$el.textContent).toBe('Hello')
  })
})
```

### Test helper functions 
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
