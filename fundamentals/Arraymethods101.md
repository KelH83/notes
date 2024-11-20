# Array-methods and higher-order functions

## Prior Knowledge

- fundamental understanding of Node
- fundamental understanding of npm
- writing tests using `jest`

## Learning Objectives

- gain understanding of higher order functions
- understand how map works
- understand how filter works

## Arrays and iteration

Consider the two functions below which create new arrays.

```js
function copyArrayAndSquare(list) {
  const output = [];

  for (let i = 0; i < list.length; i++) {
    output.push(list[i] ** 2);
  }
  return output;
}

function copyArrayAndTriple(list) {
  const output = [];

  for (let i = 0; i < list.length; i++) {
    output.push(list[i] * 3);
  }
  return output;
}

const nums = [1, 2, 3, 4, 5, 6, 7];
const squaredNums = copyArrayAndSquare(nums); // [1, 4, 9, 16, 25, 36, 49];
const tripledNums = copyArrayAndTriple(nums); // [3, 6, 9, 12, 15, 18, 21];
```

#### Generalising functionality:

- We are breaking a core principal of programming - DRY (Don't repeat yourself!)
- We can see that the two functions are almost exactly the same: they just differ in one piece of logic!
- Instead we can introduce a new function `copyArrayAndDoStuff` that takes an additional argument - a function. A function that takes another function as an argument is a **higher-order function**.
- This means we can pass in a set of instructions that does some operations and returns out a value that we can then use to build up our output array.

```js
function copyArrayAndDoStuff(list, instructions) {
  const output = [];
  // we still create a variable that hold the new values

  for (let i = 0; i < list.length; i++) {
    // here we call instructions to do something to the current
    // array element and then push the returned value from instructions to our
    // output array
    output.push(instructions(list[i]));
  }
  return output;
}
```

Now we can create a function like `tripleNumThenAdd5` and pass it to our `copyArrayAndDoStuff` which will create the new array and build up the output with the for loop. This is in effect a rough implementation of `map`.

We can see that "behind-the-scenes" map is doing several things

- it creates a new array
- iterates over the input array
- uses some function in order to build up the new array before eventually returning it

## Using map

```js
function tripleNum(num) {
  return num * 3;
}

const nums = [1, 2, 3, 4, 5];

const tripledNums = nums.map(tripleNum);
console.log(tripledNums); // [3, 6, 9, 12, 15];
```

Note how in the above example we just pass `tripleNum` to `map` in much the same way that we could pass tripleNum to `copyArrayAndSquare`. In this sense, `map` is a higher-order function.

## Using filter

```js
const nums = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

const evenNums = nums.filter(function(val) {
  return val % 2 === 0;
});

console.log(evenNums); // [2, 4, 6, 8, 10];
```

Notice how in this example `filter` is still being passed a function - except we are now passing in an **anonymous function**, seen below:

```js

function (val) {
  return val % 2 === 0;
}

```

This function only returns true or false, and is called a predicate. Filter has built into it logic that will use the return value from this function to filter elements into a new array and then return out the new array. It will _not_ amend the elements - it will return them exactly as they are.

## Pure functions

Pure functions don't have side-effects. If we use `filter` then we shouldn't be mutating variables in the global execution context, for example:

```js
const result = [];

const evens = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10].filter((num) => {
  if (num % 2 === 0) result.push(num);
});
```

This is not a good approach: we are mutating a variable in the global. `filter` itself returns a new array! So we shouldn't use the iteratee that we pass to filter to mutate variables an outer scope.

## Summary

We should use the array method that is appropriate for what we are doing. Decide on the desired outcome first, then choose a method that will achieve that goal.

- If you have an array and want a new array of the same length, with the elements transformed in some way, map will do that.
- If you have an array and want a new array with some of the elements removed, filter will do that.

There are a lot of available array methods, as always, [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array) is an excellent resource.

![a vaguely amusing representation of map, filter and reduce using food](./array-methods.jpg "a vaguely amusing representation of map, filter and reduce using food")

Credit to Steven Luscher for the original idea [(Twitter)](https://twitter.com/steveluscher)

