# Foreign Data Wrappers

FDW enable access to non-Postgres databases. Acts like an adapter or
interface to different data sources. Readonly FDWs introduced in 9.1, read/write
introduced in 9.3. Can also read non-relational data (CSV, JSON, XML).
All data will appear as a local PG table.

A FDW consists of two components:

1. Foreign table: an access method to relational/non-rel data.
2. Data link: stores link/reference to other data source and provides
   transaction semantics, file protection and read/write access to data.

FDWs are available as extensions. List: http://pgxn.org/tag/foreign%20data%20wrapper/

``CREATE FOREIGN DATA WRAPPER`` command can be used to write a custom FDW.

Set-up steps:

## Enable the FDW extension

``CREATE EXTENSION postgres_fdw;``

## Create a foreign server

Foreign server acts like an instance of an external data source,
encapsulates connection info needed by FDW to connect.

```sql
CREATE SERVER server_name FOREIGN DATA WRAPPER fdw_name OPTIONS (host
'hostname', port 'port', dbname 'my_db');
CREATE SERVER myserver FOREIGN DATA WRAPPER postgres_fdw OPTIONS (host
'foo', dbname 'foodb', port '5432');
```

## Create a user mapping (optional)

```sql
CREATE USER MAPPING FOR postgres SERVER myforeignserver OPTIONS (user
'external_user', password 'password');
CREATE USER MAPPING FOR bob SERVER foo OPTIONS (user 'bob', password 'secret');
```

## Create a foreign table

Link to the external data source (local table that must match the remote
data structure). Can use ``IMPORT FOREIGN SCHEMA`` to generate.

## Query foreign table

Foreign tables can be queried as local tables. Note that **postgres_fdw**
supports SELECT, INSERT, UPDATE and DELETE but not all operations are
supported by all FDW extensions.

# Access control

Privileges can be assigned to foreign tables like local tables. User
mapping is used for user/password to remote server. Mapped remote used
must have privileges to access remote data.

```sql
GRANT USAGE ON FOREIGN DATA WRAPPER
GRANT USAGE ON FOREIGN TABLE
```
