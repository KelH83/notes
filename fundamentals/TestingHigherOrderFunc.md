# Testing Higher order Functions

## Prior Knowledge

- Higher order functions are functions that take a function as an argument, return a function, or both.
- Understand that a function in JS is a type of object, and that additional properties can be passed to it.
- Know how to set up a testing framework with `jest`.

## Learning Objectives

- Understand why we might want to test behaviours of functions.
- Understand how to use the `arrange`, `act`, `assert` pattern to test more complicated behaviours.
- Have an awareness of how mock functions can be used to test complex behaviour.

## Testing Higher Order Functions

### Arrange, Act, Assert

Consider the higher order function `copyArrayAndDoStuff`

```js
function copyArrayAndDoStuff(list, instructions) {
  const output = [];

  for (let i = 0; i < list.length; i++) {
    output.push(instructions(list[i]));
  }

  return output;
}
```

When testing a function such as this we will will need to pass some arguments to it in order to check it behaves correctly. Frequently the invocations of our functions can be written on a single line and passed into an `expect()`. When dealing with more complicated functions however it is often beneficial to perform some setup before running the function such as below.

```js
test('instruction return values are pushed to the new array', () => {
  // arrange
  const nums = [1, 2, 3];
  const doubleNum = (num) => num * 2;
  // act
  const actual = copyArrayAndDoStuff(nums, doubleNum);
  // assert
  const expected = [2, 4, 6];
  expect(actual).toEqual(expected);
});
```

In this test we are simulating how the function will be used and this can be broken down into 3 broad steps:

1. Arrange - Setup any variables or logic you will need to test the function.
2. Act - Use the function in the way you want to test
3. Assert - perform some assertions to check the function behaved in the correct way.

When writing your tests thinking about these 3 steps can often be helpful when working out how to test the behaviours of your function.

### Testing non return value behaviours

Much of testing is concerned with the output, or `return` value of a function. However, sometimes this does not give an adequate picture of what is happening inside the function, and a more forensic analysis is needed to test these behaviours.

Consider the function `processUsers` that invokes the `sendAlert` function for each user that is a hacker.

```js
function processUsers(users, sendAlert) {
  const hackers = users.filter((user) => {
    return user.isHacker;
  });
  hackers.forEach((hacker) => {
    sendAlert(hacker);
  });
}
```

Here we don't have a return value that we can easily test. When arranging our test we will need to do some additional setup to ensure that the function we pass in will do something testable when called. An example would be testing that the `sendAlert` function we pass in is called the correct number of times.

```js
test('invokes the passed function once for each hacker', () => {
  // arrange
  const users = [
    { name: 'Mr. Robot', isHacker: true },
    { name: 'Legit user', isHacker: false },
  ];
  let hackerCount = 0;
  const countHackers = () => {
    hackerCount++;
  };
  // act
  processUsers(users, countHackers);
  // assert
  expect(hackerCount).toBe(1);
});
```

Note we are still invoking our function (`act`) but are not using the return value to test it's behaviour.
When dealing with higher order functions another common use case would be to test the correct arguments are passed to the function.

```js
test('the passed function is invoked with the hacker', () => {
  // arrange
  const users = [
    { name: 'Mr. Robot', isHacker: true },
    { name: 'Legit user', isHacker: false },
  ];
  let hackers = [];
  const collectHackers = (hacker) => {
    hackers.push(hacker);
  };
  // act
  processUsers(users, collectHackers);
  // assert
  expect(hackers).toEqual([{ name: 'Mr. Robot', isHacker: true }]);
});
```

## Mock Functions

### Creating mock functions

We can setup functions that are designed to be tested ourselves but as this is a common thing to do we can take advantage of some pre-built functions that will keep track of how they are used, what they are called with and several other useful pieces of information.

Fortunately, `jest` provides us with mock functions which can do exactly this. The documentation for them can be found [here](https://jestjs.io/docs/en/mock-functions).

Often you can just provide an empty mock function as all you need to know is how any function would be treated if it was called:

```js
// create a new mock function
const myFunc = jest.fn();

myFunc('hello');
myFunc('goodbye');

// nested arrays containing the arguments passed to the mock on each call
console.log(myFunc.mock.calls); // --> [['hello], ['goodbye']]
```

Mock functions are empty functions that return `undefined` by default. We can pass in a function when we create the mock in order to give it some specific behaviour.

```js
const doubleNumMock = jest.fn((num) => num * 2);

console.log(doubleNumMock(5)); // -> 10
```

When testing the previous `processUsers` function we could use a mock function to check we get the right behaviour instead of writing custom functions.

In this example, `sendAlertMock` will have access to the various [matchers](https://jestjs.io/docs/expect) available in jest mock function api. We can test some of these properties:

```js
const sendAlertMock = jest.fn();
const users = [{ name: 'Mr. Hacker', isHacker: true }, ...] // array of users, one of which is a 'hacker'
processUsers(users, sendAlertMock);

expect(sendAlertMock).toHaveBeenCalledTimes(1);
expect(sendAlertMock).toHaveBeenCalledWith({ name: 'Mr. Hacker', isHacker: true })
```

The first `expect` is checking how many times `sendAlertMock` was called. Our second is checking that `sendAlertMock` has been called _specifically_ with an object containing those key/value pairs. Because `toHaveBeenCalledWith` can take an arbitrary number of arguments, we can extend this test to check that `sendAlertMock` has been called with all the arguments we expect.

### Testing for Side Effects

Generally, when creating functions in JavaScript, it is a good idea to build **pure functions**.
A pure function is a function that:

- Given the same input, will always return the same output.
- Produces no side effects.

Side effects can include:

- Modifying any external variable or object property (e.g. a global variable, or a variable in the parent function scope chain)
- Logging to the console
- Writing to a file
- Writing to the network
- Triggering any external process
- Calling any other functions with side-effects

However, sometimes we have to create functions that _do_ have to cause side effects. An example could be a function that logs some output to the console. If this side effect is a key part of the functions behaviour, we should test for it.

### Using a spy to wrap an existing method

There is an example of how to spy on an existing function in the [Jest Docs](https://jestjs.io/docs/en/jest-object#jestspyonobject-methodname).

> `const spy = jest.spyOn(object, "method")` creates a spy that wraps the existing function: `object.method`. The spy will behave exactly like the original method (including when used as a constructor), but you will have access to data about all calls.

Below is an example of spying on `console.log`, and checking whether it has been invoked:

```js
test('should log hello to the console', function () {
  const consoleSpy = jest.spyOn(console, 'log');

  console.log('hello');

  expect(consoleSpy).toHaveBeenCalledTimes(1);
  expect(consoleSpy).toHaveBeenCalledWith('hello');

  consoleSpy.mockRestore();
});
```

## Jest Lifecycle Methods

Once we have set up a spy on an existing function, we will want to ensure that the amount of calls and all other properties are reset between each test. To do this, we can use the [beforeEach()](https://jestjs.io/docs/en/api#beforeeachfn-timeout) and [afterEach()](https://jestjs.io/docs/en/api#aftereachfn-timeout) methods to get Jest to run functions at certain points in the testing suite. For example, Jest will run the function passed to `beforeEach` before every subsequent `test` block.

In the example below, we wrap `console.log` in a spy before every `test` block, and remove it after every it block.

```js
describe('spying on console log', () => {
  let consoleSpy = null; // initialise variable to be accessible from all tests - its value will be reset between tests

  beforeEach(function () {
    consoleSpy = jest.spyOn(console, 'log');
  });

  afterEach(function () {
    consoleSpy.mockRestore();
  });

  it('console log has been called once', () => {
    console.log('hello');
    expect(consoleSpy).toHaveBeenCalledTimes(1);
    expect(consoleSpy).toHaveBeenCalledWith('hello');
  });

  it('console log has not been called', () => {
    expect(consoleSpy).not.toHaveBeenCalled();
  });
});
```
