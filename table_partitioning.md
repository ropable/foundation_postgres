# Table partitioning

## Partitioning

Partitioning refers to splitting a logical table into smaller physical
pieces. Query performance can be improved dramatically for certain kinds
of queries. Improved UPDATE performance - when an index no longer fits
easily in memory, both read and write operations on the index take
progressively more disk accesses.

Postgres supports basic table partitioning. It is entirely transparent
to applications.

Partitioning uses:

* Bulk deletes may be accomplished by simply removing one of the
  partitions.
* Seldom-used data can be migrated to cheaper and slower storage.
* Postgres manages partitioning via table inheritance. Each partition must be
  created as a child table of a single parent table.
* The parent table itself is normally empty; it exists just to represent
  the entire data set.
* All child tables must have the same logical attributes like columns
  and data types but can have different physical attributes like
  tablespace, fillfactor, etc.

Methods:

* Range partitioning - range partitions are defined via key column(s)
  with no overlaps or gaps.
* List partitioning - each key value is explicitly listed for the
  partitioning scheme.

## When to partition

* When a table is very large and you have reporting issues on it.
* Tables > 10 GB should be considered for table partitioning.
* Tables containing historical data where newest data can be added to a
  new partition.
* When the content of a table needs to be distributed on different
  storage devices.

## Partitioning setup

1. Create the base (master) table; this will contain no data, only defined
   constraints for all partitions.
2. Create the child or partition table(s) that each inherit from the master
   table.
3. Add table constraints to the partition tables to define the allowed
   key values in each partition.
4. Ensure that the constraints guarantee that there is no overlap
   between the key values permitted in different partitions.
5. Create indexes on each partition.
6. Optionally define a rule/trigger to redirect modifications of the
   master table to the appropriate partition.
7. Ensure that the **constraint_exclusion** config parameter is enabled
   (without this, queries will not be optimised as desired).

## Partitioning caveats

There is no way to verify that all of the CHECK constrains are mutually
exclusive. There is no way to specify that no rows should be inserted
into the master table. Each child table must be vacuumed separately if
you are using manual vacuum. In RULE-based partitioning the COPY command
cannot be used on a parent table. A FOREIGN KEY cannot refer to any
parent table column.
