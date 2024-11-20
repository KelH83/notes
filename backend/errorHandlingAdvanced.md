# Advanced Error Handling

## Learning Objectives

- Understand that tables with foreign keys may need extra error handling.
- Become more familiar with async-await syntax.
- How to use variables for columns in database query.

## Tables with foreign keys

When a table references an entry in another table, this can open up an endpoint to more room for errors.

Consider a database with a table for `species` which contains entries like 'dog', 'cat' etc, and then a table for `pets` where each pet has a species property that references the species table.

For an endpoint like `GET /api/pets?species=[speciesHere]`, where the user wants all of the pets that match the provided species, there are a few situations to consider:

**GET /api/pets?species=iDontExist** - where the species we are filtering by does _not_ exist in our database. This should give a 404 response because the resource does not exist.

**GET /api/pets?species=iguana** - here is a scenario where the species we are filtering by _does_ exist in our species table, however no pets reference this species. This should give a 200 and an empty array.

## Sending the correct response

If the database receives either of the species queries in the examples above, it will _always_ respond with an empty array. This is because, in both cases, the database has responded with all the data matching the query, aka none!

We must then differentiate between when an empty array _is_ appropriate to send back (when the query is valid but no resources associated with it) vs when we should instead send back an error (when the query is invalid).

This will require a separate request to the database to verify if the referenced resource does indeed exist in the database or if it is an invalid query. It may be useful to extract this out into its own function.

```js
const checkSpeciesExists = async (species) => {
  const dbOutput = await db.query(
    'SELECT * FROM species WHERE species_name = $1;',
    [species]
  );

  if (dbOutput.rows.length === 0) {
    // resource does NOT exist
    return Promise.reject({ status: 404, msg: 'Species not found' });
  }
};
```

If the `Promise.reject` is triggered because there is no resource found in the database, this will immediately send us to the controller's `catch` block with the 404 error object that we have created. If the rejection is _not_ triggered, then the query was valid and we should indeed send an empty array back to the user.

## Querying variable database columns

It would be good if this 'checkExists' function was reusable, as this may be a check that we want to make for different resources on our server.

It would be ideal to pass in the table and column as variables and passing those into our database query.

We can either pass these table/column variables into the query string using template literals, or make use of the `pg-format` package as below.

> **Important note:** using template literals opens the query up to SQL injection, so it is _very_ important that the table and column variables are _not_ provided by the external user. Instead, we the developers should pass in the strings for the table and column ourselves and sanitise them if necessary.

`pg-format` can be used to insert [identifiers](https://www.postgresql.org/docs/9.1/sql-syntax-lexical.html) such as table or column names into our queries using `%I`.

**Snippet 1: making checkExists reusable**

```js
// in utils.js
const format = require('pg-format');

const checkExists = async (table, column, value) => {
  // %I is an identifier in pg-format
  const queryStr = format('SELECT * FROM %I WHERE %I = $1;', table, column);
  const dbOutput = await db.query(queryStr, [value]);

  if (dbOutput.rows.length === 0) {
    // resource does NOT exist
    return Promise.reject({ status: 404, msg: 'Resource not found' });
  }
};
```

**Snippet 2: Using checkExists**

```js
// in model.js
const selectPets = async ({ species }) => {
  // getting the pets from the db here
  if (!pets.length) {
    await checkExists('species', 'species_name', species); // We, the devs, pass in the table and column name
  }
};
```
