# Loading flat files into database tables

A flat file is a text file that usually contains one record per line.
Postgres offers the ``COPY`` command to load flat files into the
database and vice versa (``COPY TO`` and ``COPY FROM`).

The default quotation character in CSV output is double-quote (").

``COPY TO`` is used to move data from a table to a file.

```sql
COPY tablename TO 'filename';
COPY customers TO '/tmp/customers.csv' WITH csv header;
```

``COPY FROM`` is used to move data from a file to a table.

```sql
CREATE TEMP TABLE cust (LIKE customers);
COPY cust FROM '/tmp/customers.csv' WITH csv header;
```

``COPY FREEZE`` is an option in the COPY statement to improve
performance of data import (violates normal rules of MVCC).
Table must be created or truncated in the current subtransaction.

```sql
COPY tablename FROM filename FREEZE;
```
