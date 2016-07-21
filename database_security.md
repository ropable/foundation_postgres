# Database security

Databases are used to store valuable, confidential data that needs
to be protected. Security is used to protect against different types of
threat. Security requirements include confidentiality, integrity and
availability. Protection includes access controls (authentication and
authorisation), data controls (views, row-level security, encryption),
network controls (SSL, firewalls) and auditing (monitoring).

Postgres provides different mechanisms for implementing security:

* Host-based access control
* Authorised access via roles
* Views
* Row level security
* Data encryption

EDB Postgres Advanced Server also includes:

* User profiles and password policy manager
* SQL/Protect (anti SQL injection)
* DBMS_CRYPTO, DBMS_RLS
* EDB-Wrap (obfuscate source code)

## Host based access control

The **pg_hba.conf** can be used to restrict the ability to connect to a
database. SSL can be forced for selected clients (hostname or IP).
Different auth methods can be used per client. Superuser access can also
be locked down to defined IPs.

## SSL

To use SSL, ``server.key`` and ``server.crt`` must be generated and kept
inside the data directory, and the ``ssl`` parameter must be set to on.
**OpenSSL** must also be installed on the server. To perform certificate
authentication, ``root.crt`` and ``root.crl`` are also required in the
data directory.

## Password authentication

MD5 or password authentication can be used, password hashes are store on
the server is ``pg_shadow``. A user can use a ``.pgpass`` file in their
Bash profile to store username & password. EDB Postgres Advanced Server
provides a password policy manager.

## LDAP Authentication

LDAP can be used to authenticate database users. Credentials are
redirected to the LDAP Server which returns the authentication result to
the Postgres server. Password policies, connection logging and auditing
can be performed on the LDAP server.

# Authorised access

## Users

Database user is separate from an OS user and a global across a database
cluster. Username must be unique and require authentication. Every
connection made to a database is made using a user. **postgres** is the
predefined superuser in the default data cluster. A different superuser
name can be specified during initialisation of a database cluster (will
have all privileges with GRANT OPTION).

``GRANT`` can be used for granting object-level privileges to database
users, roles or groups. Privs can be granted on a tablespace, database,
schema, table, sequence, domain and function. GRANT can also be used to
grant a role to a user. For help: ``\h GRANT``.

```sql
GRANT CONNECT ON DATABASE edb TO newuser;
GRANT USAGE ON SCHEMA public TO newuser;
```

``REVOKE`` can be used to revoke object privs to db users, groups or
roles. Note that ``REVOKE GRANT OPTION FOR`` can be used to revoke only
the grant option without revoking the actual privilege.

## Roles

A role is a collection of privileges. Can be created using ``CREATE
ROLE``. A privilege can be granted to a role and a role can be granted
to a user.

## Groups
A group is a collection of users. Groups can be created using the
``CREATE GROUP`` statement. A privilege can be granted to a group using
GRANT and users can be added to a group using ``ALTER GROUP``. A group
can be granted cluster and object privileges.

# Row level security
