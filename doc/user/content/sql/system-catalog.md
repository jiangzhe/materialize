---
title: "System Catalog"
description: "The system catalog stores metadata about your Materialize instance."
aliases:
  - /sql/system-tables
weight: 14
menu:
  main:
    parent: 'sql'
---

{{< version-added v0.5.0 >}}

Materialize exposes a system catalog that contains metadata about the running
Materialize instance.

{{< warning >}}
The system tables are not part of Materialize's stable interface.
Backwards-incompatible changes to these tables may be made at any time.
{{< /warning >}}

## Details

The system catalog consists of two schemas that are implicitly available in
all databases:

  * [`mz_catalog`](#mz_catalog), which exposes metadata in Materialize's
    native format.

  * [`pg_catalog`](#pg_catalog), which presents the data in `mz_catalog` in
    the format used by PostgreSQL.

These schemas contain sources, tables, and views that expose metadata like:

  * Descriptions of each database, schema, source, table, view, sink, and
    index in the system.

  * Descriptions of all running dataflows.

  * Metrics about dataflow execution.

Whenever possible, applications should prefer to query `mz_catalog` over
`pg_catalog`. The mapping between Materialize concepts and PostgreSQL concepts
is not one-to-one, and so the data in `pg_catalog` cannot accurately represent
the particulars of Materialize. For example, PostgreSQL has no notion of
[sinks](/sql/create-sink), and therefore `pg_catalog` does not display
information about the sinks available in a Materialize.

## `mz_catalog`

The following sections describe the available objects in the `mz_catalog`
schema.

### `mz_databases`

The `mz_databases` table contains a row for each database in the system.

Field  | Meaning
-------|----------
`id`   | Materialize's unique ID for the database.
`oid`  | A [PostgreSQL-compatible OID][oid] for the database.
`name` | The name of the database.

### `mz_schemas`

The `mz_schemas` table contains a row for each schema in the system.

Field         | Meaning
--------------|----------
`id`          | Materialize's unique ID for the schema.
`oid`         | A [PostgreSQL-compatible oid][oid] for the schema.
`database_id` | The ID of the database containing the schema.
`name`        | The name of the schema.

### `mz_columns`

The `mz_columns` contains a row for each column in each table, source, and view
in the system.

Field            | Meaning
-----------------|----------
`id`             | The unique ID of the table, source, or view containing the column.
`name`           | The name of the column.
`field_number`   | The index of the column in its containing table, source, or view.
`nullable`       | Can the column contain a `NULL` value?
`type`           | The data type of the column.

### `mz_indexes`

The `mz_indexes` table contains a row for each column in each index in the
system. For example, an index on `(a, b + 1)` would have two rows in this table,
one for each of the two columns in the index.

For a given row, if `field_number` is null then `expression` will be nonnull, or
vice-versa.

Field          | Meaning
---------------|----------
`id`           | Materialize's unique ID for the index.
`oid`          | A [PostgreSQL-compatible OID][oid] for the index.
`on_id`        | The ID of the relation on which the index is built.
`field_number` | If not `NULL`, specifies the column of the relation that this index column references.
`expression`   | If not `NULL`, specifies a SQL expression that is evaluated to compute the value of this index column. The expression may contain references to any of the columns of the relation.
`nullable`     | Can this column of the index evaluate to `NULL`?
`seq_in_index` | The position of this component within the index. (The order of columns in an index does not necessarily match the order of columns in the relation on which the index is built.)

### `mz_relations`

The `mz_relations` view contains a row for each table, source, and view in the
system.

Field       | Meaning
------------|---------
`id`        | Materialize's unique ID for the relation.
`oid`       | A [PostgreSQL-compatible OID][oid] for the relation.
`schema_id` | The ID of the schema to which the relation belongs.
`name`      | The name of the relation.
`type`      | The type of the relation: either `table`, `source`, or `view`.


### `mz_objects`

The `mz_objects` view contains a row for each table, source, view, sink, and
index in the system.

Field       | Meaning
------------|---------
`id`        | Materialize's unique ID for the object.
`oid`       | A [PostgreSQL-compatible OID][oid] for the object.
`schema_id` | The ID of the schema to which the object belongs.
`name`      | The name of the object.
`type`      | The type of the object: either `table`, `source`, `view`, `sink`, or `index`.

### `mz_sinks`

The `mz_sinks` table contains a row for each sink in the system.

Field          | Meaning
---------------|----------
`id`           | Materialize's unique ID for the sink.
`oid`          | A [PostgreSQL-compatible OID][oid] for the sink.
`schema_id`    | The ID of the schema to which the sink belongs.
`name`         | The name of the sink.

### `mz_sources`

The `mz_sources` table contains a row for each source in the system.

Field          | Meaning
---------------|----------
`id`           | Materialize's unique ID for the source.
`oid`          | A [PostgreSQL-compatible OID][oid] for the source.
`schema_id`    | The ID of the schema to which the source belongs.
`name`         | The name of the source.

### `mz_tables`

The `mz_tables` table contains a row for each table in the system.

Field          | Meaning
---------------|----------
`id`           | Materialize's unique ID for the table.
`oid`          | A [PostgreSQL-compatible OID][oid] for the table.
`schema_id`    | The ID of the schema to which the table belongs.
`name`         | The name of the table.

### `mz_views`

The `mz_views` table contains a row for each view in the system.

Field          | Meaning
---------------|----------
`id`           | Materialize's unique ID for the view.
`oid`          | A [PostgreSQL-compatible OID][oid] for the view.
`schema_id`    | The ID of the schema to which the view belongs.
`name`         | The name of the view.

## `pg_catalog`

Materialize has compatibility shims for the following tables from [PostgreSQL's
system catalog](https://www.postgresql.org/docs/current/catalogs.html):

  * [`pg_attribute`](https://www.postgresql.org/docs/current/catalog-pg-attribute.html)
  * [`pg_class`](https://www.postgresql.org/docs/current/catalog-pg-class.html)
  * [`pg_database`](https://www.postgresql.org/docs/current/catalog-pg-database.html)
  * [`pg_description`](https://www.postgresql.org/docs/current/catalog-pg-description.html)
  * [`pg_index`](https://www.postgresql.org/docs/current/catalog-pg-index.html)
  * [`pg_namespace`](https://www.postgresql.org/docs/current/catalog-pg-namespace.html)

These compatibility shims are largely incomplete. Most are lacking some columns
that are present in PostgreSQL, or if they do include the column the result set
its value may always be `NULL`. The precise nature of the incompleteness is
intentionally undocumented. New tools developed against Materialize should use
the documented [`mz_catalog`](#mz_catalog) API instead.

If you are having trouble making a PostgreSQL tool work with Materialize, please
[file a GitHub issue][gh-issue]. Many PostgreSQL tools can be made to work with
Materialize with minor changes to the `pg_catalog` compatibility shim.

[gh-issue]: https://github.com/MaterializeInc/materialize/issues/new?labels=C-feature&template=feature.md
[oid]: /sql/types/oid