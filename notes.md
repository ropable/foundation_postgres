# Common PostgreSQL database object names

Industry term   |   PostgreSQL term
-------------   |   ---------------
Table or Index  |   Relation
Row             |   Tuple
Column          |   Attribute
Data block      |   Page (when block is on disk)
Page            |   Buffer (when block is in memory)

# General database limits

Limit                       | Value
-----                       | -----
Max database size            | Unlimited
Max table size              | 32 TB
Max row size                | 1.6 TB
Max field size              | 1 GB
Max rows per table          | Unlimited
Max columns per table       | 25-1600 (depending on column types)
Max indexes per table       | Unlimited
