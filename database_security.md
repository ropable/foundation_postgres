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

Postgres 9.5 supports security policies for limiting access at a table
row level. By default, all rows of a table are visible. Once RLS is
enabled on a table, all queries are filtered by the security policy.
Security policies are controlled by the DBA rather than applications.
RLS offers stronger security as it is applied at the database level.

Setup using ``row_security`` config parameter. Policies:

* CREATE POLICY
* ALTER POLICY
* DROP POLICY

Enable/disable RLS for a table:

```sql
ALTER TABLE name ENABLE ROW LEVEL SECURITY;
ALTER TABLE name DISABLE ROW LEVEL SECURITY;
```

**NOTE**: table owners, superusers and roles with the ``BYPASSRLS``
attribute will bypass the RLS policy. ``FORCE ROW LEVEL SECURITY``
command can be used to force security policy on table owner. A
default-deny policy is used if no policy exists. Existing policies are
applied once RLS is enabled.

```sql
CREATE POLICY name ON tablename ....;
DROP POLICY [IF EXISTS] name ON tablename;
```

A policy can grant ability to select, insert, update or delete (or all)
for rows which match the relevant policy expression. New rows are
checked against the WITH CHECK expression. Existing rows are checked
against the USING expression.

If a dropped policy is the last policy on the table, default-deny policy
is used. It is recommended to ``DISABLE ROW LEVEL SECURITY`` on a table
after dropping all policies.

# Data encryption

## Storage level encryption

OS provide different tools for storage level encryption: full disk or
filesystem level encryption.

## Database level encryption

``pgcrypto`` provides a mechanism for encrypting selected columns. It
supports one-way and two-way encryption. Install using the CREATE
EXTENSION command. Crypto functions:

* digest - generated a binary hash
* hmac - calculates a hashed MAC for data with key
* crypt - hashing a password
* gen_salt - prepare algo parameters for crypt
* encrypt, decrypt - enc/decr data using the cipher method

# General security recommendations

* Always keep server patched (OS and software).
* Firewall the postmaster port appropriately.
* Isolate the db port from other network traffic.
* Don't rely solely on your frontend application to prevent unauthorised
  access to your db.
* Provide each user with their own login (no shared credentials).
* Allow the minimum access only.
* Use roles and classes of privileges.
* Use views and view security barriers.
* Use row-level security.
* Only allow the db superuser to log in from the server locally.
* Reserve usage of the superuser account for tasks & roles where it is
  absolutely required.
* Make as few objects owned by the superuser as necessary.
* Restrict access to config files to administrators.
* Disallow host system login by db superuser roles (postgres or
  enterprisedb).
* Disallow root login to the db server. Use personal login and then
  'sudo' to create an audit trail.
* Use a separate db login to own each database.
* Keep backups and have a tested recovery plan. Backup location should
  also be protected.
* Have scripts perform backups and immediately test them and alert a DBA
  on any failures.
* Keep backups physically separate from the db server.
* AAA verification: authenticate, authorise, audit.
