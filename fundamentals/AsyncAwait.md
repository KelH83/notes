# Async / Await

## Learning Objectives

- Know how to declare async functions
- Know how to use the `await` operator
- Know where it is possible to use the await operator
- Know how to use async/await in combination with promise chains
- Know how to wait for multiple promises to resolve using `await`
- Know the benefits and drawbacks of the async/await pattern

## Introduction

The async / await pattern is [syntactic sugar](https://en.wikipedia.org/wiki/Syntactic_sugar) which allows promise-based asynchronicity to be written in an alternative style: without promise chains, `.then` or `.catch` blocks.

## Async Functions

Async functions are declared using the `async` keyword:

```js
// function declaration
async function myFunc() {}

// arrow function
const myOtherFunc = async () => {};

// as a method in an object
const myObj = {
	method: async function () {},
};

// as a method on a class
class MyClass {
	async method() {}
}
```

> Async functions *always* return a promise. If the return value of an async function is not explicitly a promise, it will be implicitly wrapped in a promise.

For example, the following:

```js
async function returnOne() {
	return 1;
}
```

...is equivalent to:

```js
function returnOne() {
	return Promise.resolve(1);
}
```

## The `await` Operator

Once inside of an async function, the `await` operator can be put in front of any promise (or promise-based function invocation). This "pauses" execution of the code in the async function at that point until the promise fulfills. It then returns the value that the promise resolved with:

```js
async function greet() {
	const greeting = await getGreetingFromATextFile("file.txt"); // pauses here until promise is fulfilled

	console.log(greeting); // continues once greeting is fulfilled
}

greet()
```

## Error Handling in Async Functions

When using the `await` operator, handling promise rejections is best done with a `try... catch` statement:

```js
try {
	const result = await doSomethingAsync();
	// this line will not be read if doSomethingAsync rejects
	console.log(result);
} catch (err) {
	// handle error if doSomethingAsync throws or rejects
}
```

## Awaiting multiple promises

As with promise chains, [Promise.all](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all) can be used to wait for multiple promises to be fulfilled.

```js
async function doTwoAsyncThingsAtOnce() {
	const result = await Promise.all([doAsyncThing(), doAsyncThing()]);

	console.log(result); // [asyncThingValue1, asyncThingValue2]
}
```

## Mixing Async and Await with Promise Chains

Any async function returns a promise and is therefore "thenable" and can be used as part of a promise chain. Any function that returns a promise can be awaited. It is entirely feasible to mix and match promise chains with async / await style syntax. 

Take this example of an async function that takes a `fileName`, reads it using `fs/promises`, convert it to a `string` and then returns it. The function call is then chained with a `.then()` method to resolve the result of the async function call and console log it within the then block:

```js
const fs = require('fs/promises');

async function readFileAndToString(fileName) {
  const fileContents = await fs.readFile(fileName);

  return fileContents.toString();
}

readFileAndToString('greeting-text.txt').then((fileContents) => {
  console.log(fileContents);
});

```

However, it will most likely lead to more readable code when an author is consistent in their approach across a project, for example: using async/await consistently *or* using promises consistently throughout the code.

## Async Await vs Promise Chains

### Benefits of Async Await

- Can make code look synchronous (easier to read)
- Not as much need to pass variables down a promise chain because of scoping issues
- No need to always return promise (even when resolved value is not used)
- Makes control flow easier

### Disadvantages of Async Await

- Can make code look synchronous (easier to forget async nature of code)
- Not (at the time of writing) available in top-level (anywhere outside of async function) - would have to create an [IIFE](https://developer.mozilla.org/en-US/docs/Glossary/IIFE) to work around this
- `try... catch` for error handling can (arguably) become messier

## Code example of Promises vs. Async/await

In this example, we have a directory called `data/` which contains a number of files ending in different file extensions (for example, `.js`, `.txt`, `.md`). We want our function to let us know how many file(s) there are which end only in the `.js` file extension.

### `Promises` version

```js
const fs = require('fs/promises');

function getTotalNumOfJSFiles() {
  return fs
    .readdir(`${__dirname}/data`)
    .then((fileNames) => {
      const jsFiles = fileNames.filter((file) => {
        return file.endsWith('.js');
      });

      return jsFiles.length;
    })
    .catch((error) => {
      console.log('Something went wrong...');
      console.log(error);
    });
}

getTotalNumOfJSFiles().then((jsFilesTotal) => {
  console.log(
    `There are ${jsFilesTotal} JavaScript (.js) files in the 'data/' directory`
  );
});
```

### `Async/await` version

```js
const fs = require('fs/promises');

async function getTotalNumOfJSFiles() {
  try {
    const allFilesInDir = await fs.readdir(`${__dirname}/data`);

    const jsFiles = allFilesInDir.filter((file) => {
      return file.endsWith('.js');
    });

    return jsFiles.length;
  } catch (error) {
    console.log('Something went wrong...');
    console.log(error);
  }
}

getTotalNumOfJSFiles().then((jsFilesTotal) => {
  console.log(
    `There are ${jsFilesTotal} JavaScript (.js) files in the 'data/' directory`
  );
});
```

## Further Reading

- [Async functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)

- [The await operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await)

- [The try/catch statement](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/try...catch)

- [Making asynchronous programming easier with async and await](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Asynchronous/Async_await)
