# PSQL Error Handling With Express

## Prior Knowledge

- able to use `node-postgres` with an express app to query a psql database
- know some common HTTP status codes
- able to prevent unhandled promise rejections using `.catch`
- general understanding of `next` 

## Learning Objectives

- handle runtime errors in an Express application
- `.catch` psql error objects and pass them to `next`
- identify when to use custom promise rejections
- send different response status codes and messages based on different errors
- use `next` to pass errors along multiple error handling middleware functions
- extract error handling logic away from `app.js`

## 1. Error handling middleware

In express, a middleware function is a function that has access to the request object (`req`) and the response object (`res`) in the request-response cycle.

Handling errors using `console.log` or `throw` does not provide the user/client with any information as to why they have not received the expected response.

Instead of using `console.log` or `throw`, sending an error from the controller as per the below is much more useful to the end-user:

```js
if (err) res.status(500).send({ err });
```

However, having to repeat this line of code (or similar) in each controller breaks the DRY coding principle.

To fix this, the third argument that is passed by express to each middleware function and controller - **`next`** - we have seen that invoking `next()` with no argument can be used to move onto the following middleware function, but we can also use `next` to pass an error to **error-handling middleware** if we pass `next` an argument.

Error-handling middleware functions are defined in the same way as other middleware functions, except error-handling functions have **four** arguments instead of three: `(err, req, res, next)`.

The first argument of an error-handling middleware function is the error that the `next` function inside the controller is invoked with.

For example, the bellow controller will invoke the `next` function if there is an error from the `fetchPetsByOwnerId` function, which will in turn invoke the error-handling middleware function inside `app.js`:

```js
// controller.js
exports.getPetsByOwnerId = (req, res, next) => {
  const { ownerId } = req.params;
  fetchPetsByOwnerId(ownerId, (err, ownersPets) => {
    if (err) next(err);
    else res.status(200).send({ ownersPets });
  });
};

// app.js
app.get('/pets/:ownerId', getPetsByOwnerId)

app.use((err, req, res, next) => {
  console.log(err);
  res.status(500).send('Server Error!');
});
```

The benefit of being able to pass errors to specific error handling functions, it that it saves duplicate code (keeping code DRY!) and allows concerns to be separated.

Express has a default error handler for if another error-handling-middleware function has not been defined, which by default will send a status 500 with the error that `next` has been invoked with.

Error handling middleware functions attached to the Express `app` will be called before the default error handler.

## 2. Different errors and responses that should be sent

_**Note:**_ This is **NOT** a definitive list!

### GET All

- `/notARoute` -> route that does not exist: **404 Not Found**

### GET by ID

- `/api/resource/999999999` -> resource that does not exist: **404 Not Found**
- `/api/resource/notAnId` -> invalid ID: **400 Bad Request**

### POST

- `/api/resource` body: `{}` -> malformed body / missing required fields: **400 Bad Request**
- `/api/resource` body: `{ rating_out_of_five: 6 }` -> failing schema validation: **400 Bad Request**

### DELETE by ID

- `/api/resource/999999999` -> resource that does not exist: **404 Not Found**
- `/api/resource/notAnId` -> invalid ID: **400 Bad Request**

### PATCH/PUT by ID

- `/api/resource/:id` body: `{}` -> malformed body / missing required fields: **400 Bad Request**
- `/api/resource/:id` body: `{ increase_votes_by: "word" }` -> incorrect type: **400 Bad Request**



## 3. Testing and Handling Errors

Follow the normal Test Driven Development cycle (red, green, refactor) when error handling. Write a test, watch it fail:

```js
// app.test.js
// ...
test('status:400, responds with an error message when passed a bad user ID', () => {
  return request(app)
    .get('/api/users/notAnID')
    .expect(400)
    .then(({ body }) => {
      expect(body.msg).toBe('Invalid input');
    });
});
// ...
```

Once the test has failed, ensure that the controller function has a `.catch` block that will invoke `next` with any error that occurs in the promise chain:

```js
// controllers/users.js
// ...
exports.sendUserById = (req, res, next) => {
  const { user_id } = req.params;
  selectUserById(user_id)
    .then((user) => res.status(200).send({ user }))
    .catch(next);
};
// ...
```

This will pass the error to the next error handling middleware attached to the Express `app`:

```js
// app.js
// ...
app.use((err, req, res, next) => {
  console.log(err);
  res.status(500).send({ msg: 'Internal Server Error' });
});
// ...
```

### PSQL Thrown Errors

PostgreSQL throws errors which are assigned five-character error codes (**NOT the same as HTTP status codes**). [PostgreSQL Error Codes](https://www.postgresql.org/docs/current/errcodes-appendix.html)

An example psql error object:

```js
{
  error: invalid input syntax for integer: "notAnID"
    at ...
  // ...
  name: 'error',
  length: 96,
  severity: 'ERROR',
  code: '22P02',
  // ...
}
```

Based on the error that has been thrown by psql, the error handling middleware in the app can send a different response.

The below would allow relevant psql error codes to be added to the `psqlCodes` array, so that they would each cause a response to be sent with a status 400 and the appropriate psql error message.

```js
// app.js
// ...
app.use((err, req, res, next) => {
  if (err.code === '22P02') {
    res.status(400).send({ msg: 'Bad Request' });
  } else res.status(500).send({ msg: 'Internal Server Error' });
});
// ...
```

Some common instances in which psql may throw an error:

- Missing required fields (not null violation)
- Invalid input (failing schema validation)
- Column name does not exist
- Table name does not exist

### Custom Errors

In instances where psql does not throw an error, but an error status and message should be sent by the server, a custom error can be defined and thrown to the catch block for `next` to be invoked with.

An example where this may occur is when there is no error with the SQL query, but the query does not return any data/rows.

This can be identified in the model function, where the promise can be rejected with a **custom error status and message**, which would be caught by the `.catch(next)` block in the `sendUserById` controller function.

```js
// models/users.js
// ...
exports.selectUserById = (user_id) => {
  return db
    .query('SELECT * FROM users WHERE user_id = $1;', [user_id])
    .then(({ rows }) => {
      const user = rows[0];
      if (!user) {
        return Promise.reject({
          status: 404,
          msg: `No user found for user_id: ${user_id}`,
        });
      }
      return user;
    });
};
// ...
```

Once `next` is invoked with the **custom error status and message**, the error is passed to the next _error handling middleware_ function, where the appropriate HTTP status and response can be sent based on the `err` that has been received. For example:

```js
// app.js
// ...
app.use((err, req, res, next) => {
  // handle custom errors
  if (err.status && err.msg) {
    res.status(err.status).send({ msg: err.msg });
  }
  // handle specific psql errors
  else if (err.code === '22P02') {
    res.status(400).send({ msg: err.message || 'Bad Request' });
  } else {
    // if the error hasn't been identified,
    // respond with an internal server error
    console.log(err);
    res.status(500).send({ msg: 'Internal Server Error' });
  }
});
//...
```

## 4. Refactoring to Individual Error Handling Blocks

Error handling middleware functions can quickly become complex and unwieldy. To manage this, it is good practice to split the error handling logic between multiple error handling blocks, that will be responsible for dealing with different types of errors and responses.

Errors can be passed down these blocks using `next(err)` if the condition to send a specific error status and message is not met. E.g:

```js
// app.js
//...
app.use((err, req, res, next) => {
  if (err.status) {
    res.status(err.status).send({ msg: err.msg });
  } else next(err);
});

app.use((err, req, res, next) => {
  if (err.code === '22P02') {
    res.status(400).send({ msg: 'Invalid input' });
  } else next(err);
});

app.use((err, req, res, next) => {
  console.log(err);
  res.status(500).send({ msg: 'Internal Server Error' });
});
//...
```

Error handling middleware functions can be further separated into a separate file in order to keep the app tidy, and maintain all of the error handling logic in one place.

```js
// app.js
const {
  handleCustomErrors,
  handlePsqlErrors,
  handleServerErrors,
} = require('./errors/index.js');
//...
app.use(handleCustomErrors);
app.use(handlePsqlErrors);
app.use(handleServerErrors);
//...

// errors/index.js
exports.handleCustomErrors = (err, req, res, next) => {
  if (err.status && err.msg) {
    res.status(err.status).send({ msg: err.msg });
  } else next(err);
};

exports.handlePsqlErrors = (err, req, res, next) => {
  if (err.code === '22P02') {
    res.status(400).send({ msg: 'Invalid input' });
  } else next(err);
};

exports.handleServerErrors = (err, req, res, next) => {
  console.log(err);
  res.status(500).send({ msg: 'Internal Server Error' });
};
```
