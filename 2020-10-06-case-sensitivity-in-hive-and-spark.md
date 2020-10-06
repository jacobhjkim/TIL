# Case Sensitivity in Hive and Spark

> ## Hive metastore Parquet table conversion
> When reading from Hive metastore Parquet tables and writing to non-partitioned Hive metastore Parquet tables, Spark SQL will try to use its own Parquet support instead of Hive SerDe for better performance. This behavior is controlled by the spark.sql.hive.convertMetastoreParquet configuration, and is turned on by default.  
> ### Hive/Parquet Schema Reconciliation
> There are two key differences between Hive and Parquet from the perspective of table schema processing.  
> Hive is case insensitive, while Parquet is not
Hive considers all columns nullable, while nullability in Parquet is significant
Due to this reason, we must reconcile Hive metastore schema with Parquet schema when converting a Hive metastore Parquet table to a Spark SQL Parquet table. The reconciliation rules are:  
> Fields that have the same name in both schema must have the same data type regardless of nullability. The reconciled field should have the data type of the Parquet side, so that nullability is respected.  
> The reconciled schema contains exactly those fields defined in Hive metastore schema.  
> Any fields that only appear in the Parquet schema are dropped in the reconciled schema.
Any fields that only appear in the Hive metastore schema are added as nullable field in the reconciled schema.

https://spark.apache.org/docs/latest/sql-data-sources-parquet.html#hiveparquet-schema-reconciliation

TLDR
- Hive is case insensitive while Parquet is case senstive.
- This can causes some issues when your table names are in PascalCase.

That is three hours of my life that I will never get back 🤷‍♂️ But it's cool, now I know.