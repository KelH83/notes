# Error Handling Middleware and `next`

## Prior Knowledge

- Setting up an express application
- Awareness of middleware (express.json())
- MVC (Models Views Controllers)
- RESTful endpoints
- Controller functions always invoked with `req` and `res` (at the moment)
- Parametric endpoints (`req.params`) and queries (`req.query`)

## Learning Objectives

- Understand what a **middleware** function is
- Understand `next` and basic error handling

## `next`

Middleware functions are functions that have access to the request object (`req`), the response object (`res`) in the application’s request-response cycle.

One example of application-level middleware is `express.json()`, which is a function call before the routing and controller logic to deal with any possible incoming `req.body`s.

Express middleware function also have access to a third parameter: the `next` middleware function, which is commonly denoted by a variable named 'next'.

The `next` middleware function can be used to move onto the 'next' middleware function in the chain. For example the following would log the request method and time before passing control onto the following middleware function:

```js
const express = require('express');
const app = express();

app.use((req, res, next) => {
  console.log('Request method: ', req.method);
  console.log('Request url: ', req.url);
  console.log('Time:', Date.now());
  next(); // this passes the request and response onto the following middleware function
});

app.use((req, res, next) => {
  console.log('I am logged next');
  next(); // without invoking next, the request and response would not be passed onto the routes
});

app.get('/api/cats', getCats);
```

## express.json()

Express comes with some built-in middleware methods to perform common tasks. A particularly useful one for building api's is [express.json()](http://expressjs.com/en/api.html#express.json) which can be used to parse requests with JSON bodies. **nb** It must be invoked when passing it to `app.use(express.json())`.

The middleware will `JSON.parse` the request body and attach it to the request on a key of `req.body`.

When implementing methods such as `POST` or `PATCH` that commonly require bodies this is very useful. 

```js
// POST /api/users with a body of {"name": "new user", "likes": ["coding"] }

const express = require('express');
const app = express();

app.use(express.json());

app.post('/api/users', (req, res, next) => {
  console.log(req.body) // { name: 'new user', likes: ['coding'] }
  // logic for adding a new user
})
```
