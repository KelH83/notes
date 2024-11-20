# `node-postgres`

## Prior knowledge

- basic SQL statements and queries
- understanding of how JavaScript promises work
- able to access the results of promises with `.then()` and `.catch()` methods
- creating an express server with routers
- understanding of the MVC pattern

## Learning Objectives

- able to query an SQL database from a `Node.js` application
- able to set up a connection pool
- understand concept of SQL injection
- able to use dynamic database query parameters without making queries vulnerable to `SQL` injection


## 1. node-postgres

> 'node-postgres is a collection of node.js modules for interfacing with your PostgreSQL database.'

[@briancarlson, node-postgres creator](https://twitter.com/briancarlson)

## 2. Connection setup

Run `npm install pg` to install `node-postgres`.

The `node-postgres` connection can be configured within a `db` directory, alongside the SQL seeding script for the database that is to be connected to.

```
 └── node_modules/
 └── db/
 │  ├──── index.js <-- node-postgres connection pool
 │  └──── seed.sql
 ├── .gitignore
 ├── package-lock.json
 └── package.json
```

### Connecting with a single Client

A connection to a psql database is made by creating a new instance of the node-postgres `Client` class:

```js
// db/index.js
const { Client } = require("pg");

const client = new Client();

client
  .connect()
  .then(() => {
    console.log(`connected to ${process.env.PGDATABASE}!`);
  })
  .catch((err) => {
    console.log("connection error:", err);
  });
```

The database connection settings can be set using [environment variables](https://node-postgres.com/features/connecting#environment-variables):

```sh
PGDATABASE=my_database_name node db/index.js
```

Accepted environment variables include `PGDATABASE`, `PGUSER`,`PGPASSWORD`, `PGHOST`, & `PGPORT`.

**If no PGDATABASE environment variable is set, node-postgres will connect to the default database.**

### Connection Pooling

Connecting a new client to a PostgreSQL server can take 20-30 milliseconds. During this time passwords are negotiated, SSL may be established, and configuration information is shared between the client and the server. Incurring this cost every time an application wants to execute a query would substantially slow it down.

When working with web applications, or other software that makes frequent queries to a database, connection pools can be used.

A client pool allows the app to have a reusable pool of clients that can be checked-out, used, and returned. Generally an app will want a limited number of these (usually just 1), as creating an unbounded number of pools defeats the purpose of pooling at all.

[Connection pooling](https://node-postgres.com/apis/pool) with `node-postgres` follows a similar syntax to connecting with a single client, and uses the same queries api, but has some additional [config options and methods](https://node-postgres.com/apis/pool).

When there is only a single query that needs to be run on the database, the pool has a method to run a query on the first available idle client and return its result using [pool.query()](https://node-postgres.com/apis/pool#poolquery).

```js
// db/index.js
const { Pool } = require("pg");

const db = new Pool();
```

We can prevent accidental connections to the default database by checking that a database name is provided to the environment variables. It's usually useful to export the connection pool so that database queries can be run in different files:

```js
// db/index.js
const { Pool } = require("pg");

if (!process.env.PGDATABASE) {
  throw new Error("No PGDATABASE configured");
}

module.exports = new Pool();
```

### Queries

The api for executing queries supports both callbacks and promises.

Below is an example using promises:

```js
const db = new Pool();

db.query("SELECT * FROM table_name;")
  .then((result) => console.log(result))
  .catch((err) => console.log(err));
```

The [results object](https://node-postgres.com/apis/result) that is returned by `node-postgres` for a successful query has a key of `rows` which is an array of the records that have been returned by the query in the form of a json-like object for each row.


### Parameterized Queries

Passing parameters to queries can leave our code open to SQL injection. node-postgres supports [parameterized queries](https://node-postgres.com/features/queries#parameterized-query), the query method accepts a second argument of an array of values.

The indexes of the array can then be accessed in the raw sql query by using `$1`, `$2`, `$3`, etc... depending on the variables position in the array.

A query to select a specific snack from the database, could be set up as follows. 

```js
exports.selectSnackById = (snack_id) => {
  return db
    .query("SELECT * FROM snacks WHERE snack_id = $1;", [snack_id])
    .then((result) => result.rows[0]);
};
```

The above query returns an array containing a single object for the returned row.

For queries that need to be passed parameters, such as values from the request object, like `params`, `query` or `body`, **string concatenation and string template literals should NOT be used.**


## Related Resources

- [SQL injection attack YouTube video](https://www.youtube.com/watch?v=ciNHn38EyRc)
