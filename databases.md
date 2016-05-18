# Object hierarchy

![Object hierarchy](media/images/object_hierarchy.jpg)

The top-level object is the database cluster.
Databases can be owned by users or groups. Databases can use 1+
tablespaces. A database contains Catalogs/Schemas/Extensions.

# Databases

A database is a named collection of SQL objects (tables, views,
functions, etc.) It is a collection of schemas and the schemas
contain the objects.

```sql
# List databases:
\l
SELECT datname FROM pg_database;
# List users:
\du
# List tablespaces:
\db
# List schemas:
\dn
# Create/drop a database:
CREATE DATABASE name OWNER role;
DROP DATABASE name;
REVOKE CONNECT ON DATABASE name FROM public;
# Display help with syntax:
\h create database
# Connect to a database:
\c name
```

# Users

Database users are separate from OS users. Database users are global
across a db cluster. Username must be unique and requires
authentication. Every connection is made using a user. Default superuser
is **postgres** or **enterprisedb** (in Advanced Server). Default can be
changed during creation of a cluster.

```sql
# Create a user:
CREATE USER username PASSWORD 'password123' SUPERUSER;
```
