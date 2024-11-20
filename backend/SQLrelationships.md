# SQL - Foreign keys, Joins and Aggregate Functions

## Learning Objectives

- Understand different types of relations between data (one-to-one, one-to-many, many-to-many)
- Understand and know how to use different JOINS in **SQL**
- Ability to use junction tables when dealing with many-to-many relationships
- Ability to use aggregate functions to make calculations in sql queries

## Foreign Keys

A `FOREIGN KEY` is a field in one table that refers to the `PRIMARY KEY` in another table to link two tables together.

The table containing the foreign key is the child table, and the table containing the primary key is the referenced or parent table.

Foreign keys and fields representing the same kind of data on different tables should be named consistently (for example, `student_id` and `staff_id`; and `student_username` and `staff_username`, rather than `student_username` and `staff_user`).

In the below tables the `favourite_animal_id` column in the `northcoders` table is a `FOREIGN KEY` that references the `PRIMARY KEY` of the `animals` table (`animal_id`).

`northcoders`:

| northcoder_id | northcoder | favourite_animal_id |
| ------------: | ---------- | ------------------: |
|             1 | Ant        |                   2 |
|             2 | Izzi       |                   4 |
|             3 | Paul       |                   2 |
|             4 | Alex       |                   3 |

`animals:`

| animal_id | animal_name | number_of_legs |
| --------: | ----------- | -------------: |
|         1 | Spider      |              8 |
|         2 | Butterfly   |              6 |
|         3 | Unicorn     |              4 |
|         4 | Koala       |              4 |

## One-to-many Relationships

The above creates a **one-to-many** relationship between `northcoders` and `animals`; where many northcoders may have the same favourite animal.

To implement the above relationship between two tables, the `northcoders` table must be given the `favourite_animal_id` field in the `CREATE TABLE` statement. This will be of type `INT` to match the `favourite_animal_id` field in the `animals` table.

In order to have primary keys auto generated as incrementing integers, `SERIAL` can be used in place of `INT` when creating a table's primary key (e.g. `northcoder_id SERIAL PRIMARY KEY,`).

If using `SERIAL` the primary key does not need to be specified when inserting data into a table.

```sql
-- example.sql continued...

-- create tables
CREATE TABLE northcoders (
  northcoder_id SERIAL PRIMARY KEY,
  northcoder VARCHAR(40),
  favourite_animal_id INT,
  FOREIGN KEY (favourite_animal_id) REFERENCES animals(animal_id)
);
```

OR

```sql
CREATE TABLE northcoders (
  northcoder_id SERIAL PRIMARY KEY,
  northcoder VARCHAR(40),
  favourite_animal_id INT REFERENCES animals(animal_id)
);
```

## Ordering of SQL Statements

In order for the `northcoders` table to reference the `animal_id` from the `animals` table, the `animals` table must be created before the `northcoders` table, or it will throw an error from trying to reference a table, and column that does not exist.

```sql
-- example.sql
DROP DATABASE IF EXISTS northcoders_db;
CREATE DATABASE northcoders_db;

\c northcoders_db;

CREATE TABLE animals (
  animal_id SERIAL PRIMARY KEY,
  animal_name VARCHAR(40),
  number_of_legs INT
);

CREATE TABLE northcoders (
  northcoder_id SERIAL PRIMARY KEY,
  northcoder VARCHAR(40),
  favourite_animal_id INT,
  FOREIGN KEY (favourite_animal_id) REFERENCES animals(animal_id)
);
```

In order for the `northcoders` rows to be created with a reference to a specific `animal` from the `animals` table, the `animal` must exist in the `animals` table, or it will throw an error from trying to reference an `animal_id` that does not exist.

```sql
-- example.sql continued...

-- insert into tables
INSERT INTO animals
  (animal_name, number_of_legs)
VALUES
  ('Spider', 8),
  ('Butterfly', 6),
  ('Unicorn', 4),
  ('Koala', 4);

INSERT INTO northcoders
  (northcoder, favourite_animal_id)
VALUES
  ('Ant', 2),
  ('Izzi', 4),
  ('Paul', 2),
  ('Alex', 3);

-- select from tables
SELECT * FROM northcoders;
SELECT * from animals;
```

## Joins

`JOIN` clauses are used to combine records from two or more tables in a database.

There are many different Join Types in sql:

- `INNER JOIN`
  - Creates a new result table by combining column values of two tables (`table1` and `table2`) based upon the join condition.
  - The query compares each row of `table1` with each row of `table2` to find all pairs of rows, which **satisfy** the join condition.
  - When the join condition is satisfied, column values for each matched pair of rows of `table1` and `table2` are combined into a result row.
- `LEFT OUTER JOIN`
  - An inner join is performed and then, for each row in `table1` that does **not satisfy** the join condition with any row in `table2`, a joined row is added with `null` values in columns of `table2`. Thus, the joined table always has at least one row for each row in `table1`.
- `RIGHT OUTER JOIN`
  - An inner join is performed and then, for each row in `table2` that does **not satisfy** the join condition with any row in `table1`, a joined row is added with `null` values in columns of `table1`. This is the converse of a left join; the result table will always have a row for each row in `table2`.
- `FULL OUTER JOIN`
  - An inner join is performed and then, for each row in `table1` or `table2` that does **not satisfy** the join condition with any row in the other table, a joined row is added with null values in columns of the other table.
- `CROSS JOIN`
  - A CROSS JOIN matches every row of the first table with every row of the second table. If the input tables have x and y columns, respectively, the resulting table will have x+y columns. Because CROSS JOINs have the potential to generate extremely large tables, care must be taken to use them only when appropriate.

[SQL Joins Explained](https://www.geeksforgeeks.org/sql-join-set-1-inner-left-right-and-full-joins/)

An `INNER JOIN` is the most common type of join and is the default type of join.

```sql
SELECT table1.column1, table2.column2...
FROM table1
INNER JOIN table2
ON table1.common_filed = table2.common_field;
```

_**Note:** You can use the `INNER` keyword optionally._

```sql
--- example.sql continued...
SELECT *
FROM northcoders
JOIN animals
ON northcoders.favourite_animal_id = animals.animal_id;
```

The above would return the following result:

| northcoder_id | northcoder | favourite_animal_id | animal_id | animal_name | number_of_legs |
| ------------: | ---------- | ------------------: | --------: | ----------- | -------------: |
|             1 | Ant        |                   2 |         2 | Butterfly   |              6 |
|             2 | Izzi       |                   4 |         4 | Koala       |              4 |
|             3 | Paul       |                   2 |         2 | Butterfly   |              6 |
|             4 | Alex       |                   3 |         3 | Unicorn     |              4 |

## Many-to-Many Relationships

A **many-to-many** relationship may be formed when one resource will relate to one or many of another resource and those resources can relate to one or many of the other resource.

E.g. a northcoder can have many skills, and a skill can be had by many northcoders.

### A Bad Example:

Using foreign keys could lead to a table structure something like the below. However, this would require the `northcoders` table to be constantly updated if a new column was required because a northcoder had exceeded the current maximum number of skills:

`northcoders`:

| northcoder_id | northcoder | favourite_animal_id | skill_id_1 | skill_id_2 | ...more columns!? |
| ------------: | ---------- | ------------------: | ---------: | ---------: | ----------------- |
|             1 | Ant        |                   2 |          1 |       null |                   |
|             2 | Izzi       |                   4 |          3 |          3 |                   |
|             3 | Paul       |                   2 |          5 |       null |                   |
|             4 | Alex       |                   3 |          7 |          2 |                   |

`skills`

| skill_id | skill      | difficulty |
| -------: | ---------- | ---------: |
|        1 | Chess      |          7 |
|        2 | Eating     |          3 |
|        3 | Sitting    |          2 |
|        4 | Sleeping   |          1 |
|        5 | Javascript |          6 |
|        6 | HTML       |          3 |
|        7 | CSS        |         10 |

### Junction Tables

A **junction table** will allow us to create the **many-to-many** relationship, where each record in the table will refer to a relationship between a record from each of the tables being referenced.

`northcoders`

| northcoder_id | northcoder | favourite_animal_id |
| ------------: | ---------- | ------------------: |
|             1 | Ant        |                   2 |
|             2 | Izzi       |                   4 |
|             3 | Paul       |                   2 |
|             4 | Alex       |                   3 |

`skills`

| skill_id | skill      | difficulty |
| -------: | ---------- | ---------: |
|        1 | Chess      |          7 |
|        2 | Eating     |          3 |
|        3 | Sitting    |          2 |
|        4 | Sleeping   |          1 |
|        5 | Javascript |          6 |
|        6 | HTML       |          3 |
|        7 | CSS        |         10 |

`northcoders_skills`

| northcoders_skills_id | northcoder_id | skill_id |
| --------------------: | ------------: | -------: |
|                     1 |             1 |        1 |
|                     2 |             1 |        4 |
|                     3 |             1 |        7 |
|                     4 |             1 |        6 |
|                     5 |             2 |        5 |
|                     6 |             2 |        2 |
|                     7 |             2 |        1 |
|                     8 |             3 |        3 |
|                     9 |             3 |        1 |
|                    10 |             3 |        4 |
|                    11 |             4 |        5 |
|                    12 |             4 |        4 |
|                    13 |             4 |        7 |

The `northcoders_skills` table (named after the two tables that it forms a junction between) holds the information needed to store and maintain the **many-to-many** relationships between the two tables.

A junction table only needs to hold the references to the tables that it forms the **many-to-many** relationships between. Any additional information that is needed can be accessed from these tables through `JOIN` queries.

```sql
-- example.sql continued
CREATE TABLE skills (
  skill_id SERIAL PRIMARY KEY,
  skill VARCHAR(40),
  difficulty INT
);

INSERT INTO skills
  (skill, difficulty)
VALUES
  ('Chess', 7),
  ('Eating', 3),
  ('Sitting', 2),
  ('Sleeping', 1),
  ('Javascript', 6),
  ('HTML', 3),
  ('CSS', 10);

CREATE TABLE northcoder_skills (
   id SERIAL PRIMARY KEY,
  northcoder_id INTEGER REFERENCES northcoders(northcoder_id),
  skill_id INTEGER REFERENCES skills(skill_id)
);

INSERT INTO northcoder_skills
  (northcoder_id, skill_id)
VALUES
  (1,1), (1,4), (1,7), (1,6), (2,5), (2,2), (2,1), (3,3), (3,1), (3,4), (4,5), (4,4), (4,7);

SELECT * FROM northcoders_skills
JOIN northcoders ON northcoders_skills.northcoder_id = northcoders.northcoder_id
JOIN skills ON northcoders_skills.skill_id = skills.skill_id;
```

## Eliminating duplicates from queries
Depending on what columns we choose, we might get duplicate rows of data. For example, in the "skills" table above, imagine we just want to know what difficulty levels are available. 

We could do:
```sql
-- example.sql continued
SELECT difficulty from skills
ORDER BY difficulty;
```
| difficulty |
| -------------------: |
|                   1 |
|                   2 |
|                   3 |
|                   3 |
|                   6 |
|                   7 |
|                   10 |

But we don't really need to see '3' twice. Instead, we can use the `DISTINCT` keyword to eliminate duplicates:
```sql
-- example.sql continued
SELECT DISTINCT difficulty from skills
ORDER BY difficulty;
```
| difficulty |
| -------------------: |
|                   1 |
|                   2 |
|                   3 |
|                   6 |
|                   7 |
|                   10 |

It's possible to deduplcate more than one column. Suppose instead our skills table looks like this:
| skill_id | skill      | difficulty | badge |
| -------: | ---------- | ---------: | -----:|
|        1 | Chess      |          6 | silver |
|        2 | Eating     |          3 | bronze |
|        3 | Sitting    |          2 | bronze |
|        4 | Sleeping   |          1 | bronze |
|        5 | Javascript |          6 | gold |
|        6 | HTML       |          3 | bronze |
|        7 | CSS        |         10 | gold |

Now there are two activities with a difficulty of '3' and two
with '6'. We can write the following to get the list of unique
difficulty/badge combinations:
```sql
-- example.sql continued
SELECT DISTINCT difficulty, badge from skills
ORDER BY difficulty, badge;
```
The query removes duplicate values for the columns listed after the `DISTINCT` keyword. In this case, both the records with difficulty '6' are retained, because they have different badges.
| difficulty | badge |
| ---------: | -----:|
|          1 | bronze |
|          2 | bronze |
|          3 | bronze |
|          6 | gold |
|          6 | silver |
|         10 | gold |

## Aggregate Functions

There are many [aggregate functions](https://www.postgresql.org/docs/current/functions-aggregate.html) available for use in SQL. Examples include `COUNT`, `SUM`, `AVG`, `MAX`, and `MIN`. Aggregate functions perform a calculation on a set of rows and return a single row. They can be a powerful way to add extra, useful information to the results returned from a query.

In the following example, we are attempting to generate a `SUM` of the total number of legs for all the animals in the animals table.

```sql
SELECT SUM(number_of_legs) AS total_number_of_legs
FROM animals;
```

| total_number_of_legs |
| -------------------: |
|                   22 |

## Group By and Aggregate Functions

Often it is useful to query multiple tables and perform calculations on subsets of the data while being able to view other information from the queried tables. To do so, we will need to use a combination of an aggregate function and `GROUP BY`.

Consider building a query to display each animal's name and the total number of northcoders who have selected the animal as their favourite.

1. In order to to calculate this, we will need information from both tables. So, the first step in building this query is to use a `JOIN` to connect the two tables together.

```sql
SELECT *
FROM animals
JOIN northcoders ON northcoders.favourite_animal_id = animals.animal_id;
```

| animal_id | animal_name | number_of_legs | northcoder_id | northcoder | favourite_animal_id |
| --------: | ----------- | -------------: | ------------: | ---------- | ------------------: |
|         2 | Butterfly   |              6 |             1 | Ant        |                   2 |
|         4 | Koala       |              4 |             2 | Izzi       |                   4 |
|         2 | Butterfly   |              6 |             3 | Paul       |                   2 |
|         3 | Unicorn     |              4 |             4 | Alex       |                   3 |

Unfortunately, the default `JOIN` is an `INNER JOIN` which means that any records in the first table that are not references by the second table will not be included in the output of the query. `LEFT JOIN` is more appropriate here so that animals with no fans are not lost.

```sql
SELECT *
FROM animals
LEFT JOIN northcoders ON northcoders.favourite_animal_id = animals.animal_id;
```

| animal_id | animal_name | number_of_legs | northcoder_id | northcoder | favourite_animal_id |
| --------: | ----------- | -------------: | ------------: | ---------- | ------------------: |
|         2 | Butterfly   |              6 |             1 | Ant        |                   2 |
|         4 | Koala       |              4 |             2 | Izzi       |                   4 |
|         2 | Butterfly   |              6 |             3 | Paul       |                   2 |
|         3 | Unicorn     |              4 |             4 | Alex       |                   3 |
|         1 | Spider      |              8 |               |            |                     |

2. Having a row for every unique animal-northcoder combination isn't what we want. The rows created can be "squashed" together using `GROUP BY`, which will take some column to group the rows together with. Note: we can only `SELECT` columns that share the same values across the rows being grouped together.

```sql
SELECT animal_name
FROM animals
LEFT JOIN northcoders ON northcoders.favourite_animal_id = animals.animal_id
GROUP BY animals.animal_id;
```

| animal_name |
| ----------- |
| Koala       |
| Butterfly   |
| Unicorn     |
| Spider      |

At this point, the results look as if we are only querying the animals table. However, we now have access to the data in the northcoders table.

3. We are now able to use an aggregate function to count the number of northcoders associated with each animal. To do this, we count the `northcoder_id`s associated with each animal. This will show up as a new column on the result of the query called `count`.

```sql
SELECT animals.*, COUNT(northcoder_id)
FROM animals
LEFT JOIN northcoders ON northcoders.favourite_animal_id = animals.animal_id
GROUP BY animal_id;
```

4. It is possible to alias this column with `AS`:

```sql
SELECT animals.*, COUNT(northcoder_id) AS number_of_fans
FROM animals
LEFT JOIN northcoders ON northcoders.favourite_animal_id = animals.animal_id
GROUP BY animal_id;
```

| animal_id | animal_name | number_of_legs | number_of_fans |
| --------: | ----------- | -------------: | -------------: |
|         4 | Koala       |              4 |              1 |
|         2 | Butterfly   |              6 |              2 |
|         3 | Unicorn     |              4 |              1 |
|         1 | Spider      |              8 |              0 |

### ERROR: column "..." must appear in the GROUP BY clause or be used in an aggregate function

We often encounter this error in our terminal when introducing GROUP BY & Aggregate functions to our SQL. To fully appreciate why SQL will throw this error, let's take a look at the following example:

We have an imaginary table full of the following data:

`my_table`

| column_1 | column_2 |
| -------: | -------: |
|        1 |       10 |
|        2 |       15 |
|        2 |       90 |
|        1 |       10 |

We run an query on the table:

```sql
SELECT column_1, column_2 FROM my_table GROUP BY column_1;
```

In this situation we would expect all rows where `column_1 === 1` to be easy to figure out. **BUT** how do we display the data where `column_1 === 2` ?

| column_1 | column_2 |
| -------: | -------: |
|        1 |       10 |
|        2 |       ?? |

Because SQL cannot decide how to display this data, it will give an error:

`ERROR: column "my_table.column_1" must appear in the GROUP BY clause or be used in an aggregate function`

There are two ways to solve this:

1. Add `column_2` into the `GROUP BY` clause.

```sql
SELECT column_1, column_2 FROM my_table GROUP BY column_1, column_2;
```

This means that the rows will only be combined wherever the combination of the values in `column_1` and `column_2` are duplicated, giving the result:

| column_1 | column_2 |
| -------: | -------: |
|        1 |       10 |
|        2 |       15 |
|        2 |       90 |

2. Use `column_2` as part of an [aggregate function](https://www.postgresql.org/docs/current/functions-aggregate.html).

```sql
SELECT column_1, SUM(column_2) FROM my_table GROUP BY column_1;
```

Now `SQL` has some way to combine the non-unique values from `column_2`, summing them. It has no problem when producing a result:

| column_1 | column_2 |
| -------: | -------: |
|        1 |       20 |
|        2 |      105 |

## Updating existing tables

If we need to update an existing table that already contains data, we cannot just drop the table and re-create it, as this would lose all of the data contained within the table. Instead, we can use the `ALTER TABLE` statement to change the structure of an existing table.

```sql
ALTER TABLE animals
ADD COLUMN cuteness_rating INT;
```

## ON DELETE

When one table references another, we need to explicitly state what we would like to happen should the entry in the referenced table be deleted.

```sql
FOREIGN KEY(column_name)
   REFERENCES parent_table(parent_key_column)
   ON DELETE [delete_action]
```

In the case that the parent entry is deleted, two common actions are:

- `SET NULL`, which sets the value of this field to `NULL`
- `CASCADE`, which triggers the deletion of the child record as well

Other options include `SET DEFAULT`, `NO ACTION` and`RESTRICTED`.

In the following example, if one of the animals were deleted, any northcoders whose `favourite_animal_id` referenced the deleted animal would now have that field set to `NULL`.

```sql
CREATE TABLE northcoders (
  northcoder_id SERIAL PRIMARY KEY,
  northcoder VARCHAR(40),
  favourite_animal_id INT REFERENCES animals(favourite_animal_id) ON DELETE SET NULL
);
```

## Resources

A nice overview of joins can be found [here](https://stackoverflow.com/questions/38549/what-is-the-difference-between-inner-join-and-outer-join)
