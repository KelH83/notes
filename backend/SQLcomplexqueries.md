# Complex Queries

## Learning Objectives

- Be familiar with the TDD process when creating REST API endpoints
- Understand how to use aggregate functions in the context of an Express model
- Understand how to build up a complex SQL query string when dealing with optional parameters
- Know the order in which SQL statements must be written
- Know how to prevent SQL injection by manually validating input

## Optional WHERE Clauses

```js
// in a model
const queryValues = [];
let queryStr = 'SELECT * FROM table_name';

if (/* optional query value is present */) {
  queryValues.push(/* optional query value 1 */);
  queryStr += ` WHERE column_name = $1`;
}

if (/* second optional query value is present */) {
  if (queryValues.length) {
    queryStr += ' AND';
  } else {
    queryStr += ' WHERE';
  }
  queryValues.push(/* optional query value 2 */);
  queryStr += ` column_name = $${queryValues.length}`;
}
```

## LIKE patterns

When adding conditions with WHERE clauses we can use an expression that returns a boolean. Commonly we can use `=`, `>`, `<` etc but when working with strings we may frequently want to match patterns as opposed to whole strings with `=`. We can use the `LIKE` expression to match a string to a particular pattern. This can be a literal string with 2 additional wildcards:

- `_` Any character
- `%` Any number of any characters

An `I` can be added in front of the `LIKE` statement to make it case-insensitive.

```sql
'Northcoders' LIKE 'North%'      -> true
'Northcoders' LIKE '%coders'     -> true
'Northcoders' LIKE '%code%'      -> true
'Northcoders' LIKE '_orthcoders' -> true
'Northcoders' ILIKE 'north%'     -> true
```

This can then be used in models to partially match column values to a given value. The example below returns all rows where the `column_to_search` includes the `searchTerm` at any point.

```js
const queryValues = [`%${searchTerm}%`];
let queryStr = 'SELECT * FROM table_name WHERE column_to_search ILIKE $1';
```

## Ordering SQL Statements

One important consideration to make is the order in which the SQL statements are written.

The example below will error out with a fairly generic error message:

```sql
SELECT column_name(s)
FROM table_name
JOIN table_2_name
GROUP BY column_name(s) -- XXXXX
WHERE condition -- XXXXX
ORDER BY column_name(s);
```

```
ERROR:  syntax error at or near "WHERE"
LINE X: WHERE column_1_name = $1
```

However, by switching the order of the clauses (moving the `GROUP BY` clause after the `WHERE` clause), the statement will execute:

```sql
SELECT column_name(s)
FROM table_name
JOIN table_2_name
WHERE condition
GROUP BY column_name(s)
ORDER BY column_name(s);
```

## Variable ORDER BY statements

Parameters are not supported in `GROUP` or `ORDER BY` clauses within PostgreSQL. This means that whenever user input is used in these statements it could be open to SQL injection. In these situations it is **extremely important** to sanitise and input before interpolating it into the statement.

One easy way to prevent any malicious user input being added to the SQL statement is to use a "greenlist" strategy: check whether the variable matches one of a list of allowed values and reject the request if it doesn't.

```js
// `sort_by` & `order` queries from client request
if (!['allowed_column_1', 'allowed_column_2'].includes(sort_by)) {
  return Promise.reject({ status: 400, msg: 'Invalid sort query' });
}

if (!['asc', 'desc'].includes(order)) {
  return Promise.reject({ status: 400, msg: 'Invalid order query' });
}

const queryStr = `
  SELECT *
  FROM table
  ORDER BY ${sort_by} ${order};`;

// execute query
```
