# System Catelog schema

This stores info about tables and other objects. Created and maintained
automatically in ``pg_catalog``, and is always part of the
``search_path``. Contains system tables, functions and views. In
**psql**, the command ``\dS`` will return a list of the catalog of
tables and views.

```sq;
# Info about all tables:
select * from pg_tables;
# Info about all users:
select * from pg_user;
```

# System Information functions

``current_database, current_schema, version, pg_postmaster_start_time,
pg_backend_pid, inet_server_addr, inet_server_port```, etc.

```sql
select version();
select current_database();
select current_schema();
select current_user;
select pg_backend_pid();
select pg_conf_load_time();
select pg_is_in_backup();
```

# System adminstration functions

```
# Return or modify config variables:
select current_setting('search_path');
select set_config('search_path, 'otherschema', false);
# Cancel a backend's current query (requires PID):
select pg_cancel_backend(PID);
# Terminate a backend process gracefully:
select pg_terminate_backend(PID);
# Reload configuration files
select pg_reload_conf();
# Rotate a database's log files:
select pg_rotate_logfile();
# Disk space used by a tablespace, database, relation:
select pg_table_size("customers");
# Bytes used to store a particular value:
pg_column_size
# Convert a raw size to something readable:
select pg_size_pretty(pg_table_size('customers'));
```

# System information views

```sql
# Details for open connections and running transactions:
select * from pg_stat_activity;
# Details of current locks being held:
select * from pg_locks;
# Details of databases:
select * from pg_stat_database;
```
