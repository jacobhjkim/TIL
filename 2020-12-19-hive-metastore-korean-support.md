# Hive Metastore Korean Support

At PUBG all databases, tables, and columns are in English. However, the comments for the columns are sometimes in Korean. This is why I needed to modify the current Hive metastore RDS to support `utf8` rather than the default `latin1`.

---

## 1. Change RDS parameters
Here is an example terraform code for Hive metastore RDS' parameters.
Mostly I followed the steps listed here [Best practices for configuring parameters for Amazon RDS for MySQL](https://aws.amazon.com/blogs/database/best-practices-for-configuring-parameters-for-amazon-rds-for-mysql-part-3-parameters-related-to-security-operational-manageability-and-connectivity-timeout/)


```hcl
resource "aws_db_parameter_group" "aurora_db_57_parameter_group" {
  name        = "my-hive-metastore-rds-pg"
  family      = "aurora-mysql5.7"
  description = "metastore-aurora-parameter-group"

  parameter {
    name  = "init_connect"
    value = "SET NAMES utf8"
  }
}

resource "aws_rds_cluster_parameter_group" "aurora_57_cluster_parameter_group" {
  name        = "my-hive-metastore-rds-cpg"
  family      = "aurora-mysql5.7"
  description = "metastore-aurora-cluster-parameter-group"

  // https://aws.amazon.com/blogs/database/best-practices-for-configuring-parameters-for-amazon-rds-for-mysql-part-3-parameters-related-to-security-operational-manageability-and-connectivity-timeout/
  parameter {
    name  = "character_set_client"
    value = "utf8"
  }
  parameter {
    name  = "character_set_connection"
    value = "utf8"
  }
  parameter {
    name  = "character_set_database"
    value = "utf8"
  }
  parameter {
    name  = "character_set_filesystem"
    value = "utf8"
  }
  parameter {
    name  = "character_set_results"
    value = "utf8"
  }
  parameter {
    name  = "character_set_server"
    value = "utf8"
  }

  parameter {
    name  = "collation_connection"
    value = "utf8_general_ci"
  }
  parameter {
    name  = "collation_server"
    value = "utf8_general_ci"
  }
}
```

Reboot the rds instance with AWS console, then check if the changes were applied correctly.

```
mysql> show variables like 'c%';
+--------------------------+-----------------------+
| Variable_name            | Value                 |
+--------------------------+-----------------------+
| character_set_client     | utf8                  |
| character_set_connection | utf8                  |
| character_set_database   | utf8                  |
| character_set_filesystem | utf8                  |
| character_set_results    | utf8                  |
| character_set_server     | utf8                  |
| character_set_system     | utf8                  |
| collation_connection     | utf8_general_ci       |
| collation_database       | utf8_general_ci       |
| collation_server         | utf8_general_ci       |
+--------------------------+-----------------------+
```

## 2. Update current databases and tables
Since updating RDS parameters doesn't update already existing tables, I will do it manually.

[source](https://heum-story.tistory.com/34)

```sql
alter table COLUMNS_V2 modify COMMENT varchar(256) character set utf8 collate utf8_general_ci;
alter table TABLE_PARAMS modify PARAM_VALUE mediumtext character set utf8 collate utf8_general_ci;
alter table SERDE_PARAMS modify PARAM_VALUE mediumtext character set utf8 collate utf8_general_ci;
alter table SD_PARAMS modify PARAM_VALUE mediumtext character set utf8 collate utf8_general_ci;
alter table PARTITION_PARAMS modify PARAM_VALUE varchar(4000) character set utf8 collate utf8_general_ci;
alter table PARTITION_KEYS modify PKEY_COMMENT varchar(4000) character set utf8 collate utf8_general_ci;
alter table INDEX_PARAMS modify PARAM_VALUE varchar(4000) character set utf8 collate utf8_general_ci;
alter table DATABASE_PARAMS modify PARAM_VALUE varchar(4000) character set utf8 collate utf8_general_ci;
alter table DBS modify `DESC` varchar(4000) character set utf8 collate utf8_general_ci;

show full columns from COLUMNS_V2;
show full columns from TABLE_PARAMS;
show full columns from SERDE_PARAMS;
show full columns from SD_PARAMS;
show full columns from PARTITION_PARAMS;
show full columns from PARTITION_KEYS;
show full columns from INDEX_PARAMS;
show full columns from DATABASE_PARAMS;
show full columns from DBS;
```

## 3. Voilà
```
> spark.sql("DESCRIBE TABLE foo.temp").show(false)

+-----------------------+---------+---------+
|col_name               |data_type|comment  |
+-----------------------+---------+---------+
|foo                    |string   |utf-8확인 |
|dt                     |string   |날짜      |
|# Partition Information|         |         |
|# col_name             |data_type|comment  |
|dt                     |string   |날짜      |
+-----------------------+---------+---------+
```
👍