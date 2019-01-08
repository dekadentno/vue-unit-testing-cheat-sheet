# jest-cheat-sheet
:microscope: My cheat sheet for testing vue components with jest

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
