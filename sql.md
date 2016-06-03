# PostgreSQL data types

* Numeric types - numeric, integer, serial.
* Character types - char, varchar, text.
* Date/time types - timestamp, date, time, interval.
* Other - bool, money, bytea, xml, json, jsonb, blob, clob.

# Structure Query Language (SQL)

* Data definition language - CREATE, ALTER, DROP, TRUNCATE
* Data manipulation language - INSERT, UPDATE, DELETE, SELECT
* Data control language - GRANT, REVOKE
* Transaction control language - COMMIT, ROLLBACK, SAVEPOINT, SET
  TRANSACTION

```sql
# Show available data types:
\dT *  # All data types
\dT    # User-created data types only
# Show available SQL commands:
\h
```

Types of constraints: not null, check, unique, primary key, foreign key.
Constraints can be defined at the column level or table level. Can be
added to an existing table using ALTER TABLE. Can be declated deferrable
or not deferrable. Constraints can also prevent deletion if there are
dependencies (row or table).

Tables and constraints:

```sql
# Create a database and table:
CREATE DATABASE test;
\c test
CREATE TABLE test_table (
    id INTEGER PRIMARY KEY,
    article_name VARCHAR(100) NOT NULL,
    article_text TEXT
);
# Create a table with a serial column:
CREATE TABLE games (
    id SERIAL,
    name VARCHAR(100) NOT NULL,
    rating INTEGER
);
# Insert some data:
INSERT INTO games (name) VALUES ('The Witcher');
# Update some data:
UPDATE games SET name = 'The Witcher 3' WHERE id = 1;
# Delete some data;
DELETE FROM games WHERE name='The Witcher 3';
```

**Upsert** can be used to switch an insert to an update when a unique
violation occurs. To do an upsert add the ``ON CONFLICT`` clause to an
``INSERT`` statement.

# Views

A **View** is a virtual table and can be used to hide complex table
queries. Can also be used to represent a selected view of data to
restrict access to data (e.g. user has access to a view that returns a
subset of sensitive data).

```sql
CREATE OR REPLACE VIEW top_games AS SELECT * FROM games WHERE rating > 80;
```

# SQL functions

Can be used in SELECT statements and WHERE clauses. E.g.: to_char, to_number,
lower, upper, to_timestamp, etc.

```sql
SELECT lower(name) FROM departments;
SELECT * FROM departments WHERE lower(name) = 'smith';
```

# Concatenating strings

Use two vertical bars (||) to concat strings.

```
SELECT 'Department ' || department_name FROM departments;
```

# Pattern matching

Can use ``LIKE`` or regular expressions.

```sql
SELECT * FROM employees WHERE name LIKE 'ash%';
SELECT * FROM employees WHERE upper(name) LIKE 'ASH%';
SELECT * FROM employees WHERE name LIKE '%a%';
```


# Random SQL commands

```sql
# Long query (5 seconds):
SELECT pg_sleep(5);
# Current user:
SELECT user;
# Current database:
SELECT current_database();
# Port:
show port;
# Show the size of a table:
SELECT pg_size_pretty(pg_relation_size('tablename'));
# Show autovacuum parameters:
SELECT name, settings FROM pg_settings WHERE name LIKE 'autovacuum%';
```

# Lab exercise answers

```sql
select e1.ename as Employee, e1.empno as "Emp#", e2.ename as Manager,
e1.mgr as "Mgr#" from emp e1 left outer join emp e2 on (e1.mgr =
e2.empno);
select empno, ename, deptno from emp where deptno in (select deptno from
emp where lower(ename) like '%e%');
update emp set ename = 'DREXLER' where empno = 7566;
update emp set sal = 1000 where empno in (select empno from emp where
sal < 900);
```
