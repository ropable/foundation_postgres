# Extensions

Modules in the 'extension' directory are additional features to
Postgres. Generally they aren't included in the core database; they may
still be under development, tied to a PG version or not popular. Once
loaded they are available like builtin features. Extension modules are
located in ``/usr/share/postgresql/9.x/extension`` (Debian/Ubuntu).

```sql
CREATE EXTENSION module_name;
```

To make extensions available to all future database, install an
extension in the **template1** database.

## Extension module index

Several included Postgres extensions:

* Fuzzystrmatch: fuzzy string matching.
* Hstore: module for store (key, value) pairs.
* Lo: large object maintenance.
* Ltree: adds ltree data type for hierarchical treelike structure.
* pg_trgm: determine similarity of text strings.
