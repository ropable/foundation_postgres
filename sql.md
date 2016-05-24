# Command examples

```sql
# Create a database and table:
create database test;
\c test
create table test_table (
    id integer primary key,
    article_name varchar(100) not null
);
# Long query (5 seconds):
select pg_sleep(5);
# Current user:
select user;
# Current database:
select current_database();
# Port:
show port;
```
