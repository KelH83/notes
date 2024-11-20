# SQL Introduction

## Learning Objectives

- Understand the differences between relational and non-relational databases
- Be able to interact with the `psql` CLI
- Be able to create a database using `psql`
- Know the most common PostgreSQL data types
- Be able to create tables in a database
- Be able to insert data into tables
- Be able to read data in tables using conditional clauses
- Be able to update rows in tables using conditional clauses
- Be able to delete rows from tables using conditional clauses

## What is a database?

A database is a collection of information, stored electronically on a server that can be read, updated and deleted through a Database Management System (DBMS). Together, the data and the DBMS, along with the applications that are associated with them, are referred to as a database system, often shortened to just database.

There are two broad categories of databases:

- relational
- non-relational

## Relational vs. Non-Relational

A relational database is one where data is organised into tables (called "relations") with columns and rows. Each row represents a data entry, or record. Records from one table can be linked to records from another table to form relationships between the data.

A non-relational (NoSQL) database can be any type of database that doesn't use tables to organise data. NoSQL covers a range of categories of other databases, e.g. document store and key-value stores (similar to JavaScript objects).

### _Relational:_

- built from a set of **tables** (also called relations)
- data stored in **columns** and **rows** (very structured)
- a table should contain data about just one type of resource
- tables must have a **primary key** column, where each row in a table has its own unique primary key value
- when a primary key is referenced by another table, it is called a **foreign key**, this connection creates the _relationship_ between the records from multiple tables

### _Non-relational:_

- less structured data which can be more flexible
- data can be stored in documents or key-value stores
- data usually structured like json - can even be nested
- data can typically be queried by a document's unique id, or another key-value pair

### _Pros of Relational:_

- data is organised and structured. Great for when you know what your data will look like and its integrity is important. E.g. finance, retail, transaction systems
- no duplicate data
- widely used and has a lot of support, expertise and tools available
- speed of executing complex queries
- secure

### _Pros of Non-relational:_

- better for fluid data, where the requirements for data storage are likely to change
- can be easier to store and retrieve complex data
- can scale better than relational
- good for fast application development

## What is SQL?

SQL stands for **Structured Query Language** and is often pronounced "sequel".

SQL is a query language for interacting with a **relational** database.

## PostgreSQL

Although SQL does have a strict standard, there are different versions of the SQL language, which all share similar syntax and commands.

PostgreSQL (PSQL) is an _open source_ version of SQL which dates back to 1986 as part of the POSTGRES project at the University of California at Berkeley.

## psql

`psql` is a terminal based front-end to PostgreSQL, allowing a user to enter queries interactively and view the query results in the terminal.

Running the command `psql` connects to the postgres server on your computer - which runs on port 5432 by default.

Running the command `psql -l` will list all the databases stored and managed by `postgres`

There are many useful commands for interacting with a particular database:

### `psql` CLI commands

- `\l` - show databases
- `\c my_database` - connect to db called "my_database"
- `\dt` - show tables
- `\dt+` - show tables and more info
- `\d table_name` - display more information about a specific table in a database
- `\?` - show other commands
- `\q` - exit CLI
- `:q` - Return to `psql` cli from database/table/query view

## Creating a database

To drop a database (if it exists), deleting the directory containing the data, including all tables and records, the SQL statement is as follows:

```sql
DROP DATABASE IF EXISTS my_database;
```

The following SQL statement creates a database called "my_database":

```sql
CREATE DATABASE my_database;
```

Run `\c my_database` to connect to the database in the psql CLI.

To create a table inside a database, the name of the table must be specified, as well as the table's **column names** and their **data types**

## Running a file with PSQL

SQL statements can also be written in a file and then run with `psql` to make the code reusable with the following commands:

- `psql -f example.sql` to print output to the _**terminal**_
- `psql -f example.sql > example.txt` to print output to an _**example.txt**_ file

_Note: this command **cannot** be run from within the `psql` CLI_

```sql
-- example.sql
DROP DATABASE IF EXISTS my_database;
CREATE DATABASE my_database;

\c my_database

-- create tables

-- insert into tables

-- select from tables
```

## PostgreSQL **data types**:

- Boolean (`BOOL` or `BOOLEAN`)
- Character
  - `CHAR(n)` is the fixed-length character. If you insert a string that is shorter than the length of the column (n), PostgreSQL pads the spaces. If you insert a string that is longer than the length of the column, PostgreSQL will issue an error.
  - `VARCHAR(n)` is the variable-length character string. With `VARCHAR(n)`, you can store up to `n` characters. PostgreSQL does not pad spaces when the stored string is shorter than the length of the column.
  - `TEXT` is the variable-length character string. Theoretically, text data is a character string with unlimited length.
- Integers
  - `SMALLINT` is 2-byte signed integer that has a range from -32,768 to 32,767.
  - `INT` is a 4-byte integer that has a range from -2,147,483,648 to 2,147,483,647.
  - `SERIAL` is the same as integer except that PostgreSQL will automatically generate and populate values into the `SERIAL` column. This is similar to `AUTO_INCREMENT` column in MySQL or `AUTOINCREMENT` column in SQLite.
- Floating-point numbers (`FLOAT(n)`, `REAL` /`FLOAT8` `NUMERIC` or `NUMERIC(p,s)`)
- Temporal
  - `DATE` stores the date values only
  - `TIME` stores the time of day values
  - `TIMESTAMP` stores both date and time values
  - `INTERVAL` stores periods of time
- And many others! **See [PostgreSQL Data Types](https://www.postgresql.org/docs/current/datatype.html)**

## Creating a table

The following statement will create a table of northcoders, that holds information on each northcoders name. Where the primary key of each row is an `id`.

```sql
CREATE TABLE northcoders (
  northcoder_id INT PRIMARY KEY,
  northcoder VARCHAR(40) NOT NULL,
  date_of_birth DATE,
  northcoders_points INT DEFAULT 0
);
```

`NOT NULL` prevents the creation of a row without a value in the specified column.

`DEFAULT` allows a default value to be set if one is not provided when a row is added to the table.

## Inserting data into a table

Data can be inserted into the table using the following statements, inserting specific values to the columns in the table:

```sql
INSERT INTO northcoders
  (northcoder_id, northcoder)
VALUES
  (1, 'Ant');
```

Multiple records can be inserted in one statement by comma separating the entries:

```sql
INSERT INTO northcoders
  (northcoder, date_of_birth)
VALUES
  ('Ant', '1991-05-17'),
  ('Vel', null),
  ('Paul', '1990-10-24'),
  ('Alex', null);
```

## Seeding

- Database seeding is the initial process of populating a database with data. This data can be dummy data for testing or real data such as initial administrator and/or staff information.
- A seed file runs a script like a blueprint to insert data into a database.

## Querying a table for data: `SELECT`

The `northcoders` table can then be queried using a `SELECT` statement

```sql
SELECT * FROM northcoders;
```

_**Note:** `*` will return all of the columns, or column names can be specified by comma separating them_

## Syntax and Naming Conventions

### Syntax

Missing a semi-colon `;` from the end of a statement or query will result in it not being executed.

### Naming Conventions

> "There are only two hard problems in Computer Science: cache invalidation and naming things."

There are some important conventions to consider when naming tables and columns:

- Lowercase identifiers. Identifiers should be written in lower case. (e.g.: `first_name`, not `"First_Name"`)
  - This includes database names, tables and column names
  - Mixed case identifier names mean that every usage of the identifier would need to be quoted in double quotes
- Name primary keys using `[singularOfTableName]_id` format (e.g. `customer_id`).
  - If your primary key is `id` it can later be more difficult to keep track of what the `id` relates to.
- Underscores to separate words, for readability (especially given the usage of lowercase identifiers)
- Full words, not abbreviations. In general avoid abbreviations for clarity.
- Table names: plural
- Column names: singular
- Avoid reserved/key words. Do not name columns with key words such as name, new, null etc. (See: [SQL key words](https://www.postgresql.org/docs/current/sql-keywords-appendix.html))

_**Note:** the above are only suggestions. (e.g. it is still possible to write identifiers in mixed case, but this would require the use a lot of "DoubleQuotes")_

## `WHERE`, `AND`, `OR` and `NOT`

The `WHERE` clause is used to filter records to extract only those records that fulfil a specified condition.

```sql
SELECT column1, column2, ...
FROM table_name
WHERE condition;
```

The following operators can be used in the `WHERE` clause:

- `=` Equal
- `<>` Not equal. Note: In some versions of SQL this operator may be written as !=
- `>` Greater than
- `<` Less than
- `>=` Greater than or equal
- `<=` Less than or equal
- `BETWEEN` Between a certain range
- `LIKE` Search for a pattern
- `IN` To specify multiple possible values for a column

```sql
SELECT * FROM products
WHERE product_name = 'ballet shoes';
```

A `WHERE` clause with `AND` requires that two conditions are true. The following example would give you a list of all ballet shoes with a price less than or equal to 10.

```sql
SELECT * FROM products
WHERE product_name = 'ballet shoes'
AND price <= 10;
```

A `WHERE` clause with `OR` requires that one of two conditions is true.

A `WHERE` clause with `NOT` negates the specified condition. The following example would give you all of the products that have a quantity that is not 0.

```sql
SELECT * FROM products
WHERE NOT quantity = 0;
```

### WHERE LIKE

To partially match data in a `CHAR` / `VARCHAR` / `TEXT` column, we can use the `WHERE` and `LIKE` operators combined with a wildcard character, `%` or `_`. The percentage sign matches any sequence of zero or more characters. Alternatively, an underscore will match any single character. The example below will match any product that has a name ending with `" shoes"`:

```sql
SELECT * FROM products
WHERE product_name LIKE '% shoes';
```

### BETWEEN

The `BETWEEN` operator checks whether a column value falls within two provided values. The following example will match all products with a release date in the year 2022:

```sql
SELECT * FROM products
WHERE release_date
BETWEEN '2022-01-01' AND '2022-12-31';
```

### IN

The `IN` operator matches a value against a list of any number of provided values. The following example will match any products in the `"spring"` or `"summer"` seasons:

```sql
SELECT * FROM products
WHERE season IN ('spring', 'summer');
```

## ORDER BY

The `ORDER BY` clause allows query results to be sorted based on a sort expression, in a specified order. By default the order will be ascending.

```sql
SELECT * FROM products
ORDER BY rating DESC;
```

You can sort by multiple sort expressions.

```sql
SELECT * FROM products
ORDER BY rating DESC, price ASC;
```

## LIMIT

The `LIMIT` clause allows query results to be limited to a certain number of rows.

```sql
SELECT * FROM products
LIMIT 5;
```

## Specifying columns to return and aliasing with `AS`

The `AS` keyword can be used to give an alias to a column, or table name (temporarily for the current query) in order to improve readability.

```sql
SELECT product_name, season, price AS price_in_gbp
FROM products;
```

## `DELETE`

The `DELETE` statement is used to remove matching rows from a table.

The following shows basic syntax using a `WHERE` clause to specify which rows should be deleted:

```sql
DELETE FROM table_name
WHERE condition;
```

By default, the output of a `DELETE` query will be the number of affected rows (i.e. how many have been deleted).

To return the deleted row(s), a `RETURNING` clause must be included. The

```sql
DELETE FROM table_name
WHERE condition
RETURNING *;
```

The '\*' allows us to return all columns of the deleted row(s) from the table, this could be replaced with comma separated column names.

## UPDATE

The `UPDATE` statement is used to modify matching rows in a table. Combined with the `SET` keyword, this allows properties of matching records to be updated to new values.

```sql
UPDATE table_name
SET
  column1 = value1,
  column2 = column2 + value2
WHERE condition
RETURNING *;
```

## Resources

Another good tutorial site that goes into more than just the basics is [here](https://www.pgexercises.com/questions/basic/)
