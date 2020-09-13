# Hive Metastore Initialization and Upgrade

I used AWS Aurora MySQL to use it as a Hive Metastore for my my production environment. Setting up RDS in a private VPC with Terraform was relatively easy. However, connecting the RDS to Hive, Hadoop and Spark was not so easy. So here's how I did it.

### 1. `hive-site.xml`
Since I run Spark on Kubernetes make `hive-site.xml` as a Kubernetes ConfigMap.
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: hive-site
  namespace: spark-on-k8s
data:
  hive-site.xml: |
    <configuration>
      <property>
        <name>javax.jdo.option.ConnectionURL</name>
          <value>jdbc:mysql://my-metastore.rds.amazonaws.com:3306/metastore?createDatabaseIfNotExist=true</value>
      </property>
      <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
          <value>org.mariadb.jdbc.Driver</value>
      </property>
      <property>
        <name>javax.jdo.option.ConnectionUserName</name>
          <value>foo</value>
      </property>
      <property>
        <name>javax.jdo.option.ConnectionPassword</name>
          <value>bar</value>
      </property>
    </configuration>
```

### 2. Initialize Hive metastore
Clone Hive repo. [https://github.com/apache/hive](https://github.com/apache/hive/tree/master/metastore/scripts/upgrade/mysql) and run scripts under `metastore/scripts/upgrade/mysql` via accessing a bastion instance or Kubernetes pod. (I allowed my Kubernetes cluster to access the RDS.)

Using Hive metastore version `2.3.7` and cloned Hive repo under the root directory.
```bash
$ mysql -h my-metastore.rds.amazonaws.com:3306/metastore -u foo -p

mysql> USE metastore;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

mysql > source /hive/metastore/scripts/upgrade/mysql/hive-schema-2.3.0.mysql.sql;

mysql> SHOW TABLES;
+---------------------------+
| Tables_in_metastore       |
+---------------------------+
| AUX_TABLE                 |
| BUCKETING_COLS            |
| CDS                       |
| COLUMNS_V2                |
| COMPACTION_QUEUE          |
| COMPLETED_COMPACTIONS     |
| COMPLETED_TXN_COMPONENTS  |
| DATABASE_PARAMS           |
| DBS                       |
| DB_PRIVS                  |
| DELEGATION_TOKENS         |
| FUNCS                     |
| FUNC_RU                   |
| GLOBAL_PRIVS              |
| HIVE_LOCKS                |
| IDXS                      |
| INDEX_PARAMS              |
| KEY_CONSTRAINTS           |
| MASTER_KEYS               |
| NEXT_COMPACTION_QUEUE_ID  |
| NEXT_LOCK_ID              |
| NEXT_TXN_ID               |
| NOTIFICATION_LOG          |
| NOTIFICATION_SEQUENCE     |
| NUCLEUS_TABLES            |
| PARTITIONS                |
| PARTITION_EVENTS          |
| PARTITION_KEYS            |
| PARTITION_KEY_VALS        |
| PARTITION_PARAMS          |
| PART_COL_PRIVS            |
| PART_COL_STATS            |
| PART_PRIVS                |
| ROLES                     |
| ROLE_MAP                  |
| SDS                       |
| SD_PARAMS                 |
| SEQUENCE_TABLE            |
| SERDES                    |
| SERDE_PARAMS              |
| SKEWED_COL_NAMES          |
| SKEWED_COL_VALUE_LOC_MAP  |
| SKEWED_STRING_LIST        |
| SKEWED_STRING_LIST_VALUES |
| SKEWED_VALUES             |
| SORT_COLS                 |
| TABLE_PARAMS              |
| TAB_COL_STATS             |
| TBLS                      |
| TBL_COL_PRIVS             |
| TBL_PRIVS                 |
| TXNS                      |
| TXN_COMPONENTS            |
| TYPES                     |
| TYPE_FIELDS               |
| VERSION                   |
| WRITE_SET                 |
+---------------------------+
57 rows in set (0.00 sec)

```

Then boom, Hive metastore is now initialized.

### 3. Upgrade Hive metastore
Just run the upgrade script under the same directory. 👍 File names are intuitive enough.

---
### Misc
- You can run something like this. Which has the same affect, but need to install Hive and Hadoop which is a bit annoying.
```bash
hive --service schemaTool -dbType mysql -initSchema
```
- Hive metastore 3 has some cool features. Should upgrade when I have time.
