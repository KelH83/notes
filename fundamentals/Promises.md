# Promises

## Prior Knowledge

- able to identify the sources of a source of asynchronicity
- able to make asynchronous code reusable using callback functions

## Learning Objectives

- able to consume promises from 3rd party functions
- understand that promises are resolved or rejected
- understand how to promisify callback functions
- able to use Promise.all() to wait for multiple promises to resolve

## Introducing Promises

The Promise object is used for asynchronous computations. A Promise represents a placeholder for an asynchronous operation that hasn't completed yet, but is expected in the future.

Instead of immediately returning the final value, the asynchronous method returns a promise to supply the value at some point in the future. You cannot immediately access the value contained within a promise, as the asynchronous task will not have finished. The promise is said to be **pending** at this point:

```js
const promise = returnsPromise();

console.log(promise); // Promise { <pending> }
```

Promises have 3 possible states:

- **pending**: initial state, neither fulfilled nor rejected.
- **fulfilled**: meaning that the operation completed successfully.
- **rejected**: meaning that the operation failed.

Once a promise is fulfilled or rejected, its state cannot change. A promise is said to be **settled** if it is either **fulfilled** or **rejected**, but not **pending**.

Promises have `.then` method which takes a callback function as its argument, which is the only place where we can access the **fulfilled** value of the promise. This callback function is invoked on fulfilment of the promise.

```js
promise.then((response) => {
  // handle success
  console.log(response);
});
```

Promises also have `.catch` method which takes a callback function as its argument, which is the only place where we can access the error if a promise is **rejected**. This callback function is invoked on rejection of the promise.

```js
promise.catch((error) => {
  // handle error
  console.log(error);
});
```

Usually we will want to handle both possible outcomes of the promise settling, with a `.then` for if the promise is fulfilled, and a `.catch` for if the promise is rejected.

```js
promise
  .then((response) => {
    // handle success
    console.log(response);
  })
  .catch((error) => {
    // handle error
    console.log(error);
  });
```

Promises in JavaScript are non-blocking code, meaning that whilst a promise is pending, the other code in the thread will continue to be executed.

```js
promise.then(() => {
  console.log('I happen second');
});

console.log('I happen first');
```

## Benefits of using Promises

Developers will have varying opinions on the pros and cons of callbacks vs. promises. Here are some of the arguments for using promises:

- Improved readability
- Reduced nesting of code (no callback hell!)
- Error propagation - Basically, a promise chain stops if there's an exception, looking down the chain for catch handlers instead

For example, compare the below code using callbacks to create a folder and a new file within that folder, and then the subsequent example that achieves the same thing using promises:

```js
// using callbacks
const fs = require('fs');

fs.mkdir('./new-folder', (err) => {
  if (err) console.log(err);
  else {
    fs.writeFile('./new-folder/new-file.txt', 'Hello world!', 'utf8', (err) => {
      if (err) console.log(err);
      else console.log('File created!');
    });
  }
});
```

```js
// using promises
const fs = require('fs/promises');

fs.mkdir('./new-folder')
  .then(() => {
    // This function will only run if fs.mkdir is successful
    // When chaining promises together, each promise must be returned from the `.then` resolver function
    return fs.writeFile('./new-folder/new-file.txt', 'Hello world!', 'utf8');
  })
  .then(() => {
    console.log('File created!');
  })
  .catch((err) => {
    // This function will be run if either making the directory fails, or the write file fails
    console.log(err);
  });
```

Although only performing two asynchronous operations, the example using callbacks is already starting to become awkwardly nested and has two places where errors need to be handled.

The promise based example though, has a linear chain and only requires errors to be handled with a single `.catch` method. Additionally, here the `.then` resolver functions will not be called if any promise earlier in the chain is rejected.

We able to chain promises together by returning each subsequent promise from the `.then` resolver function, as each invocation of `.then` will return a promise.

## Promise.all()

`Promise.all` is a method available on the Promise constructor that allows the handling of multiple asynchronous processes.

The `Promise.all` method is passed an iterable (Array or String, but typically an array) as the only argument, and once **all** of the promises passed have resolved or when the iterable contains no promises `Promise.all` will return a single Promise that resolves with an array of values. The returned values will be in order of the Promises passed, regardless of completion order..

```js
Promise.all([promise1, promise2, promise3]).then((resolvedArray) => {
  console.log(resolvedArray); // [promise1Result, promise2Result, promise3Result]
});
```

If the iterable contains non-promise values, they will still be counted in the returned array. Below is an example of how the `Promise.all` method could be used to return the values of two different variables to a function:

```js
const word1 = 'hello';
const word2 = 'world';

Promise.all([word1, word2]).then((words) => {
  console.log(words); // ['hello', 'world']
});
```

A neat pattern for accessing the resolved values of a `Promise.all` is to use [array destructuring assignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) on the `.then` handler function's parameter:

```js
Promise.all([promise1, promise2]).then(([promise1Result, promise2Result]) => {
  // use results from promises
});
```

As `Promise.all` will return a single Promise, the `.catch` method is used in the same way to handle any promise rejections. `Promise.all` will reject immediately as soon as one of the promises is rejected

```js
Promise.all([promise1, promise2])
  .then(([promise1Result, promise2Result]) => {
    // use results from promises when all are fulfilled
  })
  .catch((err) => {
    // handle error from first rejected promise
  });
```

## Creating a Promise

Although many packages that you come across will already use Promises, there may be times where you want to refactor asynchronous code that uses callback functions to use _Promises_ instead.

In order to create a function that will return a promise, we must first understand how to create a new `Promise`.

A `Promise` object is created by using the `new` keyword and its constructor (`Promise`). The constructor takes as its argument a function, called the **"executor function"**.

The "executor function" take two functions as parameters. The first of these functions (`resolve`) is called when the asynchronous task completes successfully and returns the results of the task as a value. The second (`reject`) is called when the task fails, and returns the reason for failure (typically an error object).

```js
// ...
const returnsPromise = () => {
  return new Promise((resolve, reject) => {
    someAsyncFunction((err, data) => {
      if (err) reject(err);
      else resolve(data);
    });
  });
};
// ...
returnsPromise()
  .then((data) => {
    // do stuff with data
  })
  .catch((err) => {
    // handle error
  });
```

Consider the following example:

```js
const callbackNameAfterOneSecond = (name, callback) => {
  setTimeout(() => {
    if (!name) return callback(new Error('no name passed in!'));
    return callback(null, name);
  }, 1000);
};

callbackNameAfterOneSecond('Paul', (err, name) => {
  if (err) console.log(err);
  else console.log(`Hello ${name}!`); // logs 'Hello Paul!' after one second
});
```

We could _**promisify**_ the above function in this way:

```js
const promiseNameAfterOneSecond = (name) => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (!name) return reject(new Error('no name passed in!'));
      return resolve(name);
    }, 1000);
  });
};

promiseNameAfterOneSecond('Paul')
  .then((name) => {
    console.log(`Hello ${name}`); // logs 'Hello Paul!' after one second
  })
  .catch((err) => {
    console.log(err);
  });
```

## Resources

- [MDN - Promise Docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)

- [Using promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)

- [Promise Constructor](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/Promise)

- [Promise.all](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all)
