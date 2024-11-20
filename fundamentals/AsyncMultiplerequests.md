# Async 102 - Handling multiple requests and inversion of control

## Prior Knowledge

- Callback pattern and why callbacks are used to deal with async functions.
- How higher-order-functions work.
- Functions as first class citizens.

## Learning Objectives

- The order in which async callbacks are run is not guaranteed.
- Multiple callbacks being run at the same time must build up the result and check when all callbacks have completed before moving on.
- When dealing with arrays the results should be put back in the original index order.
- An arrays length just adds one to the maximum index, not the number of items in the array. For dealing with multiple async callbacks we need to keep a count of how many results have come in.

## Multiple asynchronous requests

In practice, we may not be dealing with a single async request: in fact, it may be that we have to send off multiple asynchronous requests and wait for them all to come back before we can continue our code.

In the example below, `getMP3` is an asynchronous function that takes a `title` and a callback function `handleMP3`. When `getMP3` has finished fetching an mp3 file, the callback function `handleMP3` is invoked with the mp3 file.

```js
getMP3('bohemian rhapsody', (err, MP3) => {
  console.log(MP3) // bohemian_rhapsody.mp3
});
```

We could also use the above function to send off multiple requests. For example, consider a situation where we wanted to get an album of mp3 files from a list of song titles.

```js
const getAlbum = (songTitles, handleAlbum) => {
  // handleAlbum will get invoked with an array of MP3s.
};
```

Normally, when writing synchronous code, we could appeal to `map` in order to create a new array of the same length: however, this no longer works because `map` uses the **return value of an iteratee**.

In this kind of problem, we need to consider several things:

1. Finding the best array method
2. Knowing when to invoke the final callback
3. Ensuring the mp3s are stored in the correct order (same as `songTitles` order)

`handleAlbum` is just a parameter name representing a callback function and these will often be named generically as `cb` or callback.

```js
const getAlbum = (songTitles, handleAlbum) => {
  const album = [];
  songTitles.forEach((songTitle) => {
    getMP3(songTitle, (err, MP3) => {
      album.push(MP3);
      if (songTitles.length === album.length) handleAlbum(null, album);
    });
  });
};
```

In the above example, `forEach` is the most appropriate array method as we are not interested in the array method's return value. `forEach` allows us to invoke an iteratee across an array: however, as the iteratee has no return value we cannot rely on `map` to create a new array.

In this situation our album array will get mutated for each callback that is invoked once the MP3 file has been retrieved. However, now we have lost the original order. However, we can pass the index to `forEach`'s iteratee and by closure when the callbacks are called they will have a reference to this index.

```js
const getAlbum = (songTitles, handleAlbum) => {
  const album = [];
  songTitles.forEach((songTitle, index) => {
    getMP3(songTitle, (err, MP3) => {
      album[index] = MP3;
      if (songTitles.length === album.length) handleAlbum(null, album);
    });
  });
};
```

The final problem is that the final callback `handleAlbum` could get called too early if the mp3 file for the last title comes back first:

```js
[undefined, undefined, undefined, undefined, undefined, 'Jump Around.MP3'];
```

Now we can introduce a counter to count the number of times each callback has been invoked:

```js
const getAlbum = (songTitles, handleAlbum) => {
  const album = [];
  let callCount = 0;
  songTitles.forEach((songTitle, index) => {
    getMP3(songTitle, (err, MP3) => {
      album[index] = MP3;
      if (++callCount === songTitles.length) handleAlbum(null, album);
    });
  });
};
```

## What is callback hell?

- Some think that **callback hell** is a term used to describe the ugly nesting that is sometimes necessary when we have to perform async tasks sequentially. It is true that that sometimes we end up with a **pyramid of doom** in our code and that this makes our code bloated and difficult to de-bug. However, with the use of callbacks, our code can also become more vulnerable to a new set of issues.

### Inversion of control

- **Inversion of control** happens when we give up control of the execution of some piece of code to a third-party. This happens all the time with callbacks : consider the following example:

```js
logIntoAccount('exampleUsername123', function(err,accountDetails) {
  if (err) // do something with errors
  else purchaseItem(accountDetails);
});
```

In the above example, `logIntoAccount` is a function from a 3rd party library. It is in charge of verifying the password and then passing the accountDetails on to the anonymous callback function.

However, in the above example, we **pass control** of the anonymous function over to the 3rd party library. It is the **3rd party library** and not ourselves that is in charge of calling this function. Therefore there is a huge trust issue involved in passing our functionality over to someone else. There could be several problems:

- the callback function could never be invoked
- the callback function could get called multiple times

In the previous example, if the callback function gets called multiple times then we could end up with `purchaseItem` being invoked multiple times and thus charging the user's credit card more than once.

We can use a variable stored in the closed over variable environment closure to control the number of times the code **inside** the callback gets executed.

```js
let hasBeenCalled = false;

logIntoAccount('exampleUsername123', function(err,accountDetails) {
  if (!hasBeenCalled) {
    hasBeenCalled = true;
    if (err) // do something with errors
    else purchaseItem(accountDetails);
  }
});
```
