# Seeding from JSON with node-postgres and pg-format

## Prior Knowledge

- Basic SQL Syntax
- JavaScript Promises
- Queries with `node-postgres`

## Learning Objectives

- Create tables in a PostgreSQL database programmatically
- Insert JSON data to a PostgreSQL database programmatically
- Handle references within data by writing util functions using TDD

## Seeding databases

In order to insert complex and often larger quantities of data into our databases it makes more sense for us to do this programmatically rather than with manual data entry, this is even more necessary if the data we are entering is already stored on a computer in the form of a .csv .json or some other standard file type.

### Creating and dropping tables in order

The order in which tables are created and dropped in SQL is important when it comes to foreign keys. For a table to reference another table that table must already exist which means we must create our tables in the right order. For example if we have 3 tables:

- table_1 - no references
- table_2 - references table_1
- table_3 - references table_2

The these tables must be created in that order. e.g.

```
CREATE TABLE table_1 (...);
CREATE TABLE table_2 (...);
CREATE TABLE table_3 (...);
```

The reverse applies when dropping tables. As table_1 is referenced by table_2, we must drop table_2 before dropping table_1. e.g.

```
DROP TABLE IF EXISTS table_3;
DROP TABLE IF EXISTS table_2;
DROP TABLE IF EXISTS table_1;
```

## Using node-postgres to insert JSON data

Previously we have used raw SQL scripts to initially populate our databases. However, we will often have data that needs to be inserted into our database that would be time-consuming to manually add to a SQL script. Similar to how we are able to create databases programmatically using `node-postgres`, we can also insert data using the `query` method.

Using `node-postgres` we can insert a record to a table using a [parameterised query](https://node-postgres.com/features/queries#parameterized-query).

However, if and when we have different numbers of records that need to be inserted (for example, less records in our test data than our development data) this becomes more challenging, as we would need to dynamically build up the SQL string, by looping through the data and keeping track of the query parameter positions for the parameterized query.

To help solve this problem, we can use [`pg-format`](https://www.npmjs.com/package/pg-format), which is recommended by the `node-postgres` documentation to safely create dynamic SQL queries in a similar way to the [PostgreSQL format function](https://www.postgresql.org/docs/9.3/functions-string.html#FUNCTIONS-STRING-FORMAT).

To use `pg-format` to insert multiple values, we can pass two arguments:

- An SQL query string, which can contain a placeholder for the formatted values
- A nested array of the values to be inserted for each record

`%L` in the example below acts as a placeholder for where the values will be inserted into the string

```js
const format = require('pg-format');
const db = require('./');

const itemsInsertStr = format(
  `INSERT INTO items
    (item_name, quantity)
  VALUES
    %L
  RETURNING *;`,
  nestedArrOfValues // [['item-A', 3], ['item-B', 5], [...], ...]
);

db.query(itemsInsertStr).then((itemsInsertResult) => {
  //
});
```

## Handling data with relationships when seeding

An integral part of SQL databases is that it is relational, so it is highly likely that we will need to insert data into one table that references entries in another.

In order to reference a specific database entry at insertion, it is necessary to have its primary key. Primary keys are frequently generated at the point of insertion, such as with `SERIAL` types. This means it important to consider the order that the data is inserted so that the values in the parent table can be generated before being referenced.

With the primary keys from the parent table, the child data may need to be manipulated to include these values. In the example below, for each book, the `book_genre` property would need to be swapped for a `genre_id` property before insertion. When manipulating the data ready for seeding, be mindful that you are not mutating the original data.

```js
const genres = [
  { genre: 'thriller' },
  { genre: 'romance' },
  { genre: 'sci-fi' },
];

const books = [
  { title: 'bookA', book_genre: 'thriller' },
  { title: 'bookB', book_genre: 'thriller' },
  { title: 'bookC', book_genre: 'romance' },
];
```

## Extracting seeding logic

Seeding functions are made up of several distinct pieces of functionality. Writing the logic for creating and dropping tables as well as the data formatting and insertions can often lead to large and hard to read functions. A more extensible approach is to extract some of the logic into dedicated functions to make the seed more readable. Consider the example above for seeding an items table.

We can extract the logic of dropping and creating tables to their own dedicated functions. Consider the following example with the table logic extracted to a `manage-tables.js` file.

```js
// db/manage-tables.js
exports.dropTables = () => {
  return db.query(`DROP TABLE IF EXISTS items;`);
};

exports.createTables = () => {
  return db.query(
    `CREATE TABLE items (
      item_id SERIAL PRIMARY KEY,
      item_name VARCHAR NOT NULL,
      quantity INT NOT NULL
    );`
  );
};

// db/seed.js
const { dropTables, createTables } = require('./manage-tables.js');

const seed = ({ itemData }) => {
  return dropTables()
    .then(() => {
      return createTables();
    })
    .then(() => {
      // insert data
    });
};
```

**nb** Make sure that your functions `return` promises so that you an chain them together using `.then` blocks
