# Async 101

## Prior knowledge

- Understand the call stack, thread and variable environment
- Be able to follow the thread of execution in synchronous code
- Understand what closure is and how to use it
- Higher order functions that take functions as arguments

## Learning Objectives

- Learn why Node was invented and the difference between blocking and non-blocking I/O
- Understand the asynchronous nature of JavaScript, including how runtime works with the Node API, task queue and the event loop
- Learn how to use callback functions in order to work with the result of an async function

## I/O

**I/O** stands for input / output and refers to any process by which our computer's CPU (central processing unit) communicates with an external device. Inputs are the signals or data received by the CPU and outputs are the signals or data sent from it. A typical **I/O operation** could be a user entering an **input** on a keyboard and an output being some text appearing on the screen. Another typical **I/O operation** could be when we ask the computer (input) to read something from disk (output).

### Blocking I/O

When developing applications that handle many different users we want to avoid as much as possible **blocking I/O** operations. **Blocking I/O** operations are ones in which the thread is effectively held up whilst waiting for the I/O operation to complete. This means that with a particularly time consuming operation it is impossible for the thread to be executing any other tasks. When trying to build applications that deal with multiple users doing lots of different requests, blocking I/O operations can make the application far less performant. Consider the example below:

```js
const doMoreWork = () => {
  console.log('doing more work...');
};

const blockFor3Seconds = () => {
  const startTime = Date.now();
  while (Date.now() < startTime + 3000) {}
};

blockFor3Second();
doMoreWork();
```

Our code is read linearly, so `doMoreWork()` will not be executed until after the while loop inside `blockFor3Seconds` ends. `blockFor3Seconds` is not a useful function at all - instead, think of it as representing another I/O operation like reading something from computer memory or a request to another server for some data. If this process is **blocking** then we are unable to go ahead and run any more code in the main thread of execution.

### Non-blocking I/O

Ryan Dahl, the creator of Node, set out to create a javascript runtime (an environment where we can execute code) that makes use of **non-blocking I/O**.

Consider the following example:

```js
const doMoreWork = () => {
  console.log('doing more work...');
};

function doTask1() {
  console.log('doing task 1...');
}

setTimeout(doTask1, 3000);
doMoreWork();
```

When we run the above example, we can see something different from the up to now synchronous execution of code. `doMoreWork()` is executed and then later on `doTask1` is invoked. This is called **non-blocking** code. To be clearer, Node has done something with our `doTask1`, and will come back to run it at a later point in time. Node has carried on reading the rest of the code in the thread in the meantime.

Currently we understand our code with:

- The thread of execution
- The variable environment (VE)
- The call stack

In order to understand async code to the same degree, we're going to have to add some additional sections to our conceptual model. They are as follows:

- Browser / Node API (background threads)
- callback / Task Queue
- The event loop

#### Application Programming Interface

An application programming interface (API) is a set of tools, definitions and protocols that a developer can use for interacting with a piece of software. If we talk about an facebook's API then we are describing a bunch of methods and tools that are exposed to developers for interacting with Facebook's data. The term Node API can be used to describe the tools, definitions and protocols that can be used when interacting with Node.

#### Background Threads

When `setTimeout` is invoked it is passed 2 arguments - a **function** and a number representing a time in milliseconds. This **function** isn't going onto our call stack, otherwise it would be invoked straight away. `setTimeout` is actually storing the **function** internally within Node - this area gives us part of Node's API and we needn't be concerned with the exact location of this place in Node's architecture. The most important part of this area is that it handles **background threads** - these are processes that are being run in the background until they have been processed as complete.

```js
const doMoreWork = () => {
  console.log('doing more work...');
};

function doTask1() {
  console.log('doing task 1...');
}

setTimeout(doTask1, 3000);
doMoreWork();
```

The function `doTask1` we have passed to `setTimeout` is stored against the timer of `3000` on the Node API. The background thread running it can now keep checking the timer, and when it is finished, enqueue it onto the **task queue**.

#### Task/Callback Queue

The **task/callback queue** is a queue data structure, which is FIFO (meaning first in first out). In other words, the first function into the task queue will be the first to be invoked later on.

In the context of our example above, when 3000ms is up, the callback `doTask1` is passed into the **task/callback queue** and patiently waits for it's turn to be invoked.

#### Event Loop

The **event loop** is responsible for constantly checking the **call stack** to see if all the tasks on the stack have been completed. If so, it is then responsible for allowing the next callback function on the **task queue** to be pushed onto the **call stack**. At this point our callback function re-enters the JS runtime and will be executed by Node (i.e. back to synchronous land again).

With respect to the above example, once all of the tasks in the global environment have completed (the `doMoreWork` function will be the last task to complete) our `doTask1` callback (which is currently waiting in the **task queue**), will be allowed onto the **call stack** by our **event loop**.

Consider also the example below:

```js
const doMoreWork = () => {
  console.log('doing more work...');
};

function doTask1() {
  console.log('doing task 1...');
}

setTimeout(doTask1, 0);
doMoreWork();
```

Even though the requested time to run `doTask1` is 0ms, the **event loop** still has to wait until all the tasks in the global environment have been completed before it can pass `doTask1` back into the JS runtime to be invoked.

#### Run to completion

Consider also the example below:

```js
const doMoreWork = () => {
  console.log('doing more work...');
};

function doTask1() {
  console.log('task 1: starting ...');
  console.log('task 1: finishing...');
}

function doTask2() {
  console.log('task 2: starting ...');
  console.log('task 2: finishing...');
}

setTimeout(doTask1, 5);
setTimeout(doTask2, 5);
doMoreWork();
```

Even though we have `doTask1` & `doTask2` callbacks, which would both be ready at roughly the same time, the **event loop** would not attempt to flip between the execution of the two functions. `doTask2` would have to wait in the **task queue** until the execution of `doTask1` was completed and closed before allowing `doTask2` to be invoked.

This is the principle of **running to completion**. The task currently running must run up to completion before the next task is run by the event loop.

## Sources of asynchronicity

There are 5 main sources of asynchronous code:

1. User interactions
2. Network I/O
3. Disk I/O
4. IPC - Interprocess communication i.e. web workers
5. Timers - `setTimeout` can be included here

We can consider some of these sources of asynchronicity in more depth:

### Disk I/O

#### `fs`

Node has a built-in module called `fs`, an API for accessing the computer's file system. It allows you to read, write and delete files amongst other things. Consider the example below that makes use of the `readFileSync` function:

```js
const { readFileSync } = require('fs');

const myQuote = readFileSync('./quote.txt', 'utf8');
doMoreWork();
```

In the above example, we are saving the return value of `readFileSync` into a variable called `myQuote`. Nothing else can be done in the thread of execution until `readFileSync` has finished reading the file. In other words, the thread won't execute `doMoreWork()` until the file has been read and `readFileSync` has returned a string. This is a synchronous way of working and it means that at the moment we are **blocking** the main thread of execution.

Compare the above example with the example below:

```js
const { readFile } = require('fs');

readFile('./quote.txt', 'utf8', (err, quote) => console.log(quote));
doMoreWork();
```

The above example illustrates what is known as [continuation passing style](https://medium.com/@b.essiambre/continuation-passing-style-patterns-for-javascript-5528449d3070)
**Continuation passing style** means that where a function would normally return, there is instead a **callback function** that contains the rest of the program execution. The **callback function** will be invoked at a later point, after `doMoreWork()`, in order to continue any functionality with the quote.

### Anatomy of callback function

```js
(err, quote) => {
  // logic involving quote goes here...
};
```

By convention all callback functions in Node are written error-first: that is to say, our functions will be written with an `err` parameter first. This way if something goes wrong, the callback function will be invoked with an error argument informing us what went wrong with the async call. For example,

```js
const { readFile } = require('fs');

readFile('./asasdt.txt', 'utf8', (err, quote) => {
  if (err) console.log(err);
  else console.log(quote);
});
doMoreWork();
```

In the example above, `err` is not null as the file `'./asasdt.txt'` doesn't exist. We see `err` as an object that looks like this in our console output:

```raw
{ [Error: ENOENT: no such file or directory, open './quote-library/quote99.txt']
  errno: -2,
  code: 'ENOENT',
  syscall: 'open',
  path: './asasdt.txt' }
```

The `err` object above tells us that `readFile` cannot find a file given the the file path argument.

## Network I/O

Another common source of asynchronicity you will experience is the interaction between your computer and another computer on a network. An example of this could be when we request data from a server on the internet, this interaction will naturally take time and depend on the time it takes for the remote server to respond. This time it takes for a message to travel across a network is called **latency**. Naturally, we want this process to be **non-blocking** so the thread can be executing other tasks whilst the external server processes our request.

### Making requests

There are a range of different 3rd party libraries we can use for making requests across a network. We will consider in the following examples a function called `request` that makes requests across the internet to fetch some data. It is important to note that there are **many ways** of doing non-blocking requests across the network other than using `request`. In the following example, it makes requests to the following [address](https://mini-dog-server.herokuapp.com/api). `request` takes a path to indicate what data we want from the given address. If we make a request to `/dogs`, for example, then we would expect a response that it is an array of dog objects.

Consider the erroneous example below:

```js
const request = require('./utils/request');

const allDogs = request('/dogs');
console.log(allDogs);
```

If we were to approach this problem from a synchronous perspective, then we might be expecting `request` to **return** something and for us to save the **return value** of `request` into a variable. This would mean that `request` is blocking the thread of execution. Therefore we cannot use the above approach.

Instead, as `request` is an asynchronous function, we can pass an error-first callback function as an argument to `request`. When the request has completed, the callback function will be added to the task queue where it will wait for the call stack to be empty for its turn to be invoked/ pushed onto the call stack by the event loop.

Suppose now that we want to create a function called `fetchDogs` that basically wraps the logic of our call to `request`:

```js
const request = require('./utils/request');

const fetchDogs = () => {
  request('/dogs', (err, response) => {
    console.log(response);
  });
};
```

But we face one huge issue: once invoked how do we continue to work with the data from our request. We could put a `return` statement inside the callback function: however, `fetchDogs` would still not return anything. In fact, the only place where we now have access to the `response` variable is within the scope of the callback function. So we could do the following:

```js
const request = require('./utils/request');

const logDogs = (err, dogs) => {
  if (err) console.log(err);
  else console.log(dogs);
};

const fetchDogs = () => {
  request('/dogs', (err, response) => {
    if (err) logDogs(err);
    else logDogs(null, response.dogs);
  });
};
```

Instead of returning anything, we have a function that is passed the result from the async function in order to continue working with the response from the `request`. What happens, however, if we instead want to save the dogs in a file for later use.

```js
const request = require('./utils/request');
const { writeFile } = require('fs');

const logDogs = (err, dogs) => {
  if (err) console.log(err);
  else console.log(dogs);
};

const saveDogs = (err, dogs) => {
  if (err) console.log(err);
  else {
    writeFile('./dog-kennel.json', JSON.stringify(dogs), (err) => {
      console.log('doggies in the kennel....');
    });
  }
};

const fetchDogs = () => {
  request('/dogs', (err, response) => {
    if (err) saveDogs(err);
    else saveDogs(null, response.dogs);
  });
};
```

This is however, impractical, if we want to generalise this function `fetchDogs` then we must allow our function to take a callback function as an argument. This way we execute whatever functionality we like with the dogs by passing in a callback function to `fetchDogs` as an argument.

```js
const request = require('./utils/request');
const { writeFile } = require('fs');

const logDogs = (err, dogs) => {
  if (err) console.log(err);
  else console.log(dogs);
};

const saveDogs = (err, dogs) => {
  if (err) console.log(err);
  else {
    writeFile('./dog-kennel.json', JSON.stringify(dogs), (err) => {
      console.log('doggies in the kennel....');
    });
  }
};

const fetchDogs = (callback) => {
  request('/dogs', (err, response) => {
    if (err) callback(err);
    callback(null, response.dogs);
  });
};

fetchDogs(logDogs);
fetchDogs(saveDogs);
```

Most of the time, however, we will pass the callback functions directly into the async functions as anonymous arrow functions, as in the example below:

```js
const request = require('./utils/request');
const { writeFile } = require('fs');


const fetchDogs = (callback) => {
  request('/dogs',(err , response) => {
    if (err) callback(err);
    else callback(null, response.dogs);
  });
};

fetchDogs((err,dogs) => {
  if (err) console.log(err);
  else {
    writeFile('./dog-kennel.json', JSON.stringify(dogs), err => {
      console.log('doggies in the kennel....');
    });
  }
};

```

## Summary

Node is a run-time environment for javascript that allows us to run javascript outside of the browser and to perform non-blocking I/O asynchronous operations. We can use callback functions as means of carrying on the execution of the code after our asynchronous task has finished, meanwhile the main thread of execution can continue to work on other tasks.
