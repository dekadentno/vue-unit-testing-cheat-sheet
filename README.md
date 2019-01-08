# jest-cheat-sheet
:microscope: My cheat sheet for testing vue components with jest

Useful links: 
* [jest documentation](https://jestjs.io/docs/en/getting-started)
* [globals](https://jestjs.io/docs/en/api)
* [expect](https://jestjs.io/docs/en/expect)

## Setuping jest 
Follow these steps for the most basic jest setup:
1. ```npm i --save-dev jest```
2. add ```"unit": "jest"``` to your package.json file
3. create ```Component.spec.js``` file in the component folder
4. Write tests in the ```Component.spec.js``` file and run it with ```npm run unit```

### Sanity test
```javascript
describe('Component.vue', () => {
  test('sanity test', () => {
    expect(true).toBe(true)
  })
})
```
