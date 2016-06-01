# Maintenance

Data files become fragmented as data is modified/deleted. Maintenance
should be proactive. Improves performance and saves db from transaction
ID wraparound failures. An update/delete of a row does not immediately
remove the row from the disk page. Eventually this row space becomes
obsolete and causes fragmentation/bloat.

Maintenance commands: **ANALYZE, VACUUM, CLUSTER**. Maintenance command
``vacuumdb`` can be run from the command prompt.

Optimizer statistics play a vital role in query planning, but are not
updated in real time. Collect information for relations inluding size,
row counts, average row size and row sampling. Stats are stored
permanently in catalog tables. The maintenance command ``ANALYZE``
updates the stats (e.g. after a bulk operation). To update:

```sql
ANALYZE tablename;
```
