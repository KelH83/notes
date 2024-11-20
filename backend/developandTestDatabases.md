# Databases for Testing and Development

## Prior Knowledge

- JavaScript Promises
- node postgres seeding

## Learning Objectives

- Be able to use `dotenv` to organise environment variables
- Reasons for separate test and development databases
- Connect to different PostgreSQL databases programmatically

## Setting environment variables

`node-postgres` finds which database to connect through from the environment variables. Until now, we have set the name of the database manually in our `package.json` scripts (`PGDATABASE=name-of-database-here`). As is often the case in JavaScript, there is an `npm package` that can help us to do this.

`dotenv` is an [npm package](https://www.npmjs.com/package/dotenv) that handles the configuration of environment variables. Once it is installed, we can create a file called `.env` and store our variables in there. An important benefit of this is that we can `gitignore` this file so that we do not push any sensitive information about our database and/or postgres passwords up to github.

```txt
PGDATABASE=your-database-name-here
```

Then, in the /db file that we set up our connection to the database we can require `dotenv` into the file and invoke its `config` method. This sets all of the environment variables from the `.env` file to the `process.env`.

```js
// in our db file where we set up our connection to the database
require('dotenv').config();
const { Pool } = require('pg');

module.exports = new Pool();
```

Note that if, for some reason, the PGDATABASE environment variable was not set correctly, `node-postgres` would connect to your default database rather than throwing an error. This could cause lots of issues if not spotted early, and so it is advisable to add a condition that checks the `process.env.PGDATABASE` variable exists and throws an error if missing.

```js
if (!process.env.PGDATABASE) {
  throw new Error('No PGDATABASE configured');
}
```

## Separate `test` and `development` databases

Running tests can cause updates to the data within a database, which can make the state of the data in the database unpredictable across tests. Re-seeding the database before running the test suite allows the data used for the tests to be more predictable and consistent.

However, if the same database is used for user interaction with the application (for example using a browser or Insomnia), the user would experience unexpected changes to the data when the test database was re-seeded, or if the tests made changes to the data. This is why it is beneficial to have a separate database for running test scripts that can be regularly re-seeded, and another for manual use of the application where any changes made by a user can persist.

An added benefit of this is that the test data can be curated by the developers to be a smaller, more manageable subset of the development data. This means that test data can be re-seeded quicker and is easier to update and write tests for.

## Connecting to different PostgreSQL databases

In order to use `node-postgres` to connect to different databases, the `db/index.js` file needs to be configured to programmatically change which database we are connecting to based on the `process.env.NODE_ENV`.

Up until now, we have stored the database name in a `.env` file and used the `dotenv` package to read the this file and set the environment variable of `PGDATABASE` to the `process.env`. Now that we have different databases to connect to depending on the environment ('development' vs 'test'), we will need multiple .env files.

In `.env.test`

```
PGDATABASE=test_database_name
```

In `.env.development`

```
PGDATABASE=development_database_name
```

In the `db/index.js` file itself, we need to configure `dotenv` so that it can find the correct file depending on our node environment (`process.env.NODE_ENV`). If the `process.env.NODE_ENV` is `test`, then `dotenv` needs to get the variables from the `.env.test` file, and if the `NODE_ENV` is `development` then `.env.development`.

As a default, `dotenv` will look for a file simply called `.env`. If the filename is something different, as in our case, we can pass an object (which specifies the path to the correct file) into the `config` method on the required-in `dotenv` module.

```js
const { Pool } = require('pg');

require('dotenv').config({
  path: pathToCorrectEnvFileHere,
});

module.exports = new Pool();
```

To find the path to the correct `.env` file, we have to a) work out the name of the file, and b) create an absolute path to that file to pass into the config object.

To find the name for the file, the naming convention for our `.env.test` and `.env.development` means we can append the `process.env.NODE_ENV` to `.env`.

Creating the absolute path to this file is slightly more work-intensive. This is because there is a disconnect between the relative path from the `db/index` file to the correct `.env` file, versus where the script that runs the project will actually be executed (likely in the root of the project).

The below example uses the globally available [`__dirname`](https://nodejs.org/api/modules.html#modules_dirname) to create an absolute path to the correct `.env` file. This allows the file to be run from anywhere within the project directory structure and maintain an absolute path to the correct `.env` file.

```js
// /Users/username/project-folder/db/index.js
const { Pool } = require('pg');
const ENV = process.env.NODE_ENV || 'development';

const pathToCorrectEnvFile = `${__dirname}/../.env.${ENV}`;

console.log(__dirname); // /Users/username/project-folder/db
console.log(pathToCorrectEnvFile); // /Users/username/project-folder/.env.development

require('dotenv').config({
  path: pathToCorrectEnvFile,
});

module.exports = new Pool();
```

Remember that if the PGDATABASE environment variable is not set, we will be connected to our _default_ database which could cause issues, so it is still advisable to add a condition that checks the `process.env.PGDATABASE` variable exists and throws an error if missing.

## Making a reusable `seed` function

A `seed` function is one that will remove any existing data from a database, recreate the tables and repopulate it dynamically. Consider the example below that does this for a table of items with the raw item data being held in a data folder.

```js
// db/seed.js
const format = require('pg-format');
const db = require('./index.js');

const seed = ({ itemData }) => {
  return db
    .query(`DROP TABLE IF EXISTS items;`)
    .then(() => {
      return db.query(
        `CREATE TABLE items (
          item_id SERIAL PRIMARY KEY,
          item_name VARCHAR NOT NULL,
          quantity INT NOT NULL
        );`
      );
    })
    .then(() => {
      const queryStr = format(
        `INSERT INTO items
          (item_name, quantity)
          VALUES
          %L
          RETURNING *;`,
        itemData.map((item) => [item.item_name, item.quantity])
      );
      return db.query(queryStr);
    });
};

module.exports = seed;
```

This same function could then be used to both seed the test database and development database.

For the test database, we can require our `seed` function in and invoke it with the correct data inside the `beforeEach`. This will ensure that before _each_ test, the database is re-seeded, solving our problem of unpredictable data if its changed between tests. `jest` will set the `process.env.NODE_ENV` to `test` for us, meaning the correct database will be connected to and seeded.

```js
// /Users/username/project-folder/__tests__/app.test.js
const seed = require('../db/seed');
const testData = require('../db/data/test-data');

beforeEach(() => seed(testData));
```

For the development database, we can create a file that will run our seed for us, with the development data. Because we are not setting the `process.env.NODE_ENV` here, it'll default to `development` and connect to that database.

```js
// /Users/username/project-folder/db/seed-dev.js
const data = require('./data/dev-data');
const seed = require('./seed');

const db = require('./');

seed(data).then(() => {
  return db.end();
});
```
