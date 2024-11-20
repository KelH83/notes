# Using `node-postgres` with `express` 

### GET example

#### Express app setup

Install `express`.

Setup `app.js` and `snacks.js` controllers:

```js
// app.js
const express = require('express');
const app = express();
const { sendSnacks } = require('./controllers/snacks.js');

app.get('/api/snacks', sendSnacks);

module.exports = app;
∏
// controllers/snacks.js
exports.sendSnacks = (req, res) => {
  res.status(200).send('GET all snacks route OK.');
};
```

#### Accessing and sending data from the database

Each controller should be responsible for invoking a model that will provide data in the format that is to be sent to the client.

As such, it is the model function that will need to use the `client` that is exported from `db/index.js` to query the database.

```js
// models/snacks.js
const db = require("../db/index.js");

exports.selectSnacks = () => {
  return db.query("SELECT * FROM snacks;").then((result) => result.rows);
};
```

The model function must `return` the query so that the result can be accessed in the `.then()` block when the model is invoked in the controller function.

By returning `result.rows` from the query, the data is sent to the controller function as an array, ready to be sent to the client. Making the controller function extremely succinct.

```js
// controllers/snacks.js
const { selectSnacks } = require("../models/snacks.js");

exports.sendSnacks = (req, res) => {
  selectSnacks().then((snacks) => res.status(200).send({ snacks }));
};
```

```
 ├─── node_modules/
 ├─── controllers/
 │  └──── snacks.controllers.js
 ├─── db/
 │  ├──── index.js <-- node-postgres connection pool
 │  └──── seed.sql
 ├── models/
 │  └──── snacks.models.js
 ├── app.js
 ├── index.js
 ├── .gitignore
 ├── package-lock.json
 └── package.json
```

## `node-postgres` parameterized query examples

### GET by ID example

For a request to `GET` a specific snack from the database, the endpoint could be set up as follows:

#### Route and Controller Setup

```js
// app.js
app.get("/api/snacks/:snack_id", sendSnackById);

// controllers/snacks.js
exports.sendSnackById = (req, res) => {
  const { snack_id } = req.params;
  selectSnackById(snack_id).then((snack) => res.status(200).send({ snack }));
};
```

#### Model Setup

For an endpoint that uses values from the request object, such as `params`, `query` or `body`, **string concatenation and string template literals should NOT be used.**

This can (and often does) lead to _**sql injection**_ vulnerabilities. This means that external code, written by the client, could be run on the database server, potentially giving access to read, write and modify sensitive data (e.g. passwords, email addresses, personal data, etc.)

The model function in this case would then need to use the `snack_id` parameter in the SQL query,.

Without having to use a template literal, or string concatenation, the query method accepts a second argument of an array of values.

The indexes of the array can then be accessed in the raw sql query by using `$1`, `$2`, `$3`, etc... depending on the variables position in the array.

```js
// models/snacks.js
exports.selectSnackById = (snack_id) => {
  return db
    .query("SELECT * FROM snacks WHERE snack_id = $1;", [snack_id])
    .then((result) => result.rows[0]);
};
```

The above query returns an array containing a single object for the returned row.

The resulting row should be destructured before being returned to the controller. This is because the model in the MVC framework should be responsible for both fetching and formatting the data ready for the controller to send to the client.

### POST example

For a request to `POST` a new snack to the database, the endpoint could be set up as follows.

#### Route Setup

The `.post()` method can be added to the existing routes in `app.js`

```js
// app.js
app.post("/api/snacks", addSnack);
```

#### Controller Setup

The controller is then responsible for passing the request body to the model, and sending the newly inserted snack to the client.

```js
// controllers/snacks.js
exports.addSnack = (req, res) => {
  insertSnack(req.body).then((snack) => res.status(201).send({ snack }));
};
```

#### Model Setup

Within the model, the key values for the data that is being inserted can be destructured from the body that has been sent, and passed into the array that `.query()` accepts as its second argument.

`'...RETURNING *;'` must be added to the end of the `'INSERT INTO...'` SQL statement in order for the newly inserted row to be returned by the query.

```js
// model/snacks.js
exports.insertSnack = (newSnack) => {
  const { snack_name, price, flavour_text } = newSnack;
  return db
    .query(
      "INSERT INTO snacks (snack_name, price, flavour_text) VALUES ($1, $2, $3) RETURNING *;",
      [snack_name, price, flavour_text]
    )
    .then(({ rows }) => rows[0]);
};
```

## Related Resources

- [SQL injection attack YouTube video](https://www.youtube.com/watch?v=ciNHn38EyRc)
