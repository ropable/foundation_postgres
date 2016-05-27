# Authentication and authorisation

Secure access is a 2 step process: authentication (ensures that a user
is who they claim) and authorisation (ensure that an auth'd user has
access to only the data for which they have permission).

Levels of security:

* Server and application - checks client IP, pg_hba.conf
* Database - user/password, connect privilege, schema permissions
* Object - table level privileges, grant/revoke

# Access Control

The host-based access control file is ``pg_hba.conf`` in the data
directory of a cluster. Any changes required reload. Each record
specifies connection type, database name, user name, client IP and
method of authentication (trust, reject, md5, password, etc.) Records
are read from top to bottom.

Access records are in the format:

```
TYPE DATABASE USER IPADDRESS METHOD
local all postgres           trust
```

Note that there are also some settings in ``postgres.conf`` that control
application access to a cluster:

* list_addresses
* max_connections
* superuser_reserved_connections
* port
* unix_socket_directory
* unix_socket_group
* unis_socket_permissions

# Row level security

GRANT and REVOKE can be used at the table level. PostgreSQL 9.5 supports
security at the row level via policies:

```sql
ALTER TABLE accounts ENABLE ROW LEVEL SECURITY;
CREATE POLICY account_managers ON accounts TO managers USING (manager = current_user);
CREATE POLICY user_policy ON users USING (user = current_user);
```

# Privileges

Users need ``CONNECT`` privileges on a database, ``USAGE`` on a schema
and ``SELECT`` on a table or view. See [Databases](databases.md).
