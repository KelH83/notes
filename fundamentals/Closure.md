# Closure

## Prior Knowledge

- Higher order functions are functions that either take a function as an argument or return a function.
- An understanding of how variables are looked up in JavaScript according to the rules of scope.
- Understanding that JavaScript is a lexically scoped language

## Learning Objectives

- Learn how functions can be returned out of functions
- Understand what closure means in javascript
- Learn how to use closure to create functions with certain useful behaviours

## Garbage collector

In previous examples, we have considered what happens with the call stack, thread and the variable environment.

Consider the example below:

```js
function incrementCounter() {
  let count = 0;
  return ++count;
}

incrementCounter();
incrementCounter();
```

The function `incrementCounter` is invoked in two different places. On each call a new local execution context is created and a local variable `count` is declared. When the return statement is reached the execution context is closed. JavaScript has a mechanism under the hood called the garbage collector, which comes along and removes any variables from memory that are not being used anymore. JavaScript does this automatically but this is not the case in other languages. In the example above, once the local execution context for `incrementCounter` has been closed then the `count` variable is removed from memory by the garbage collector.

In this way `incrementCounter` is a misnomer as its return value is always fixed as 1. It would be desirable to have a function that would instead return consecutive numbers on each sucessive call.

## Returning Functions

Consider the example below:

```js
function makeCounter() {
  function innerCounter() {
    let count = 0;
    return ++count;
  }
  return innerCounter;
}

const counter = makeCounter();
counter();
counter();
```

In this example, we have a function `makeCounter` and its **return value** is a **new function**. `makeCounter` is said to be a **higher order function** as it returns a function. We are assigning a reference to this function to `counter`. Every time this new function `counter` is invoked a new execution context is created and a new local counter is created as in the previous example.

However, we can make a subtle yet highly important change to the previous example in the following way:

```js
function makeCounter() {
  let count = 0;
  function innerCounter() {
    return ++count;
  }
  return innerCounter;
}

const counter = makeCounter();
counter();
counter();
```

In the above example, `count` is declared in the outer function `makeCounter`. `makeCounter` then returns the `innerCounter` function and `makeCounter`'s local execution context is closed: however, the `innerCounter` function itself now **references a variable in the parent scope**.

Despite the fact that the `innerCounter` is exposed to the global, because of lexical scope the function `innerCounter` can still reference the variables in the outer scope which means the `count` variable is not garbage collected in this example. This is an example of **closure** in javascript.

**Definition:** Closure is when a function is able to access its outer lexical environment (where it was declared) even when that function is executed elsewhere. An inner function will always have access to the parent function's variable even after the parent function's local execution context has closed.

## Using Closure

Consider the next example below:

```js
function createTimesTwo() {
  function multiplyBy2(n) {
    return n * 2;
  }
  return multiplyBy2;
}

const doubleNum = createTimesTwo();
const result = doubleNum(50);
```

1. We create a function called `createTimesTwo` in the global VE
2. We create a variable called `doubleNum` in the global VE. Its value is currently `undefined` as it is being assigned to a function invocation. In other words, a local execution context will be opened and the return value of `createTimesTwo` will be saved into the variable `doubleNum`
3. The return value of `createTimesTwo` is a function, namely a reference to the function `multiplyBy2`
4. We are saving the return value of `doubleNum` invoked with an argument of 50, into the variable `result`

We can adapt the example we have just written above in order to exploit closure.

Here is a slightly different function `createMultiplier`:

```js
function createMultiplier(mult) {
  function multiplyNum(num) {
    return num * mult;
  }
  return multiplyNum;
}

const multiplyBy5 = createMultiplier(5);
const result = multiplyBy5(9);
```

1. We create a function called `createMultiplier` in the global variable environment (VE)
2. We create a variable called `multiplyBy5` in the global VE. Its value is currently `undefined` as it is being assigned to a function invocation. As with the previous example the return value of `createMultiplier` will be assgined to `multiplyBy5`.
3. When the local execution context for `createMultiplier` is closed the variable `mult` is not garbage collected as it is being referenced by `multiplyNum`. This is closure - where the inner function's reference to the outer function's variables still persists, even if this inner function is returned out of `createMultiplier` and exposed to the global.
4. The variable `mult` is said to be stored in a COVE, standing for closed over variable environment.
5. When `multiplyBy5` is invoked later on it attempts to reference `mult` and closure means it will look for this variable in the COVE. Therefore as `mult` has been assigned a value of 5 the variable `num` in `multiplyNum` will always be multipled by 5.

This example shows how we can use closure to create a more generic function that can create a function that could multiply by any number - dependent on what we pass as an argument to `createMultipler`.

## Enhanced functions

A particular use of closure could be to create new functions whose behaviour is restricted or controlled in some way. Suppose we wanted a new function `add3TimesOnly` which will only return a sum a **limited** number of times. Any subsequent calls to `add3TimesOnly` would return `undefined`.

```js
add3TimesOnly(10, 32); // returns 42
add3TimesOnly(4, 20); // returns 24
add3TimesOnly(50, 100); // returns 150
add3TimesOnly(34, 17); // returns undefined
```

So the `add3TimesOnly` function can only return the sum **three times**. Lets consider how to generate such a function:

```js
const add = (a, b) => a + b;

function createLimitedFunc(func, maxCalls) {
  function limitedFunc(a, b) {
    return func(a, b);
  }
  return limitedFunc;
}
const add3TimesOnly = createLimitedFunc(add, 3);

add3TimesOnly(10, 32);
add3TimesOnly(4, 20);
add3TimesOnly(50, 100);
add3TimesOnly(34, 17);
```

We can break down what is happening in the above example:

1. `add` and `createLimitedFunc` are functions stored in the global variable environment
2. `add3TimesOnly` is assigned to an invocation of `createLimitedFunc`. The value of `add3TimesOnly` is temporarily undefined until we have got the return value of `createLimitedFunc`.
3. `createLimitedFunc` is invoked with 2 arguments, `func` will point to `add` and `maxCalls` will now have a value of `3`.
4. In the local execution context for `createLimitedFunc`, the function `limitedFunc` is declared and then returned. The local execution context for `createLimitedFunc` is closed.

In the above example, the return value of `add3TimesOnly` is the return value of `func`. In other words, whatever `func` returns is what `add3TimesOnly` or `limitedFunc` returns. In this way the `limitedAdd` function behaves exactly like the `add` function. However, there is no limit at the moment to how many times the `limitedFunc` can return `func(a,b)`. We can introduce some conditional logic into the `limitedFunc` in order to ensure that the `limitedFunc` only returns the `func(a,b)` as many times as the passed `maxCalls`. We can see this below:

```js
const add = (a, b) => a + b;

function createLimitedFunc(func, maxCalls) {
  function limitedFunc(a, b) {
    if (--maxCalls >= 0) return func(a, b);
  }
  return limitedFunc;
}
const limitedAdd = createLimitedFunc(add, 3);

limitedAdd(10, 32);
limitedAdd(4, 20);
limitedAdd(50, 100);
limitedAdd(34, 17);
```

Every time the `limitedFunc` is invoked then the `maxCalls` that is stored in the COVE gets decremented and we can only return `func(a, b)` provided the `maxCalls` is bigger.

## Other resources:

- [Master the JS Interview: What is a Closure? by Eric Elliot](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-closure-b2f0d2152b36#.szp9dswih)
- [Learn JavaScript Closures with Code Examples by Preethi Kasireddy](https://www.freecodecamp.org/news/lets-learn-javascript-closures-66feb44f6a44)
- [W3Schools: JavaScript Function Closures](https://www.w3schools.com/js/js_function_closures.asp)
