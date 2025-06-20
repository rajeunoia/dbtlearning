# dbtlearning
Repo to store DBT learnings for future reference

Models 

CTE - Common Table expressions 

DBT Power User

Materialization
View 
Table
Incremental 
Ephemeral

Seeds - Loading data from CSV
Copy the file to SEEDS folder 
Run a command dbt seed and it will load SEED files into Tables 

Sources 
source.yml

Snapshots
SCD Type 2 implementation of DBT (dbt_valid_from, dbt_valid_null)
Strategies: Timestamp -> Uniqie key + Updated_at or Check --> set of columns or All columns



|| Config || Description & allowed values || Spark || Glue || Databricks ||
| file_format | Storage format (‘parquet’, ‘delta’, ‘hudi’, ‘iceberg’, …) | ✓ | ✓ | ✓ |
| location_root | Root path for managed/external tables | ✓ | ✗ | ✓ |
| partition_by | Column(s) to partition the table | ✓ | ✓ | ✓ |
| clustered_by | Column(s) to bucket (requires buckets) | ✓ | ✗ | ✓ |
| buckets | Number of buckets when clustering | ✓ | ✗ | ✓ |
| tblproperties | Key-value table properties | ✓ | ~ | ✓ |
| options | Extra USING OPTIONS key-values | ✓ | ✗ | ✗ |
| custom_location | Glue – S3 path override | ✗ | ✓ | ✗ |
| table_type | Glue – ‘hive’ | ‘iceberg’ | ✗ | ✓ | ✗ |
| format | Glue – storage-format shorthand | ✗ | ✓ | ✗ |
| ha | Glue – high-availability swap build | ✗ | ✓ | ✗ |
| s3_data_naming | Glue – prefix naming strategy | ✗ | ✓ | ✗ |
| versions_to_keep | Glue – how many HA versions to retain | ✗ | ✓ | ✗ |
| lf_tags_config | Glue – Lake-Formation tags | ✗ | ✓ | ✗ |
| timeout | Glue – job timeout (seconds) | ✗ | ✓ | ✗ |
| spark_encryption / spark_requester_pays / spark_cross_account_catalog | Glue work-group flags | ✗ | ✓ | ✗ |
| databricks_compute | Target SQL Warehouse / compute | ✗ | ✗ | ✓ |
| submission_method | ‘all_purpose_cluster’ | ‘job_cluster’ | ✗ | ✗ | ✓ |
| cluster_id / job_cluster_config | Explicit cluster settings | ✗ | ✗ | ✓ |
| create_notebook | Emit compiled code as a notebook | ✗ | ✗ | ✓ |
| zorder | Z-order Delta table on column(s) | ✗ | ✗ | ✓ |
| incremental_strategy | append | insert_overwrite | merge | … | ✓ | ✓ | ✓ |
| incremental_predicates | Predicates list for replace_where | ✗ | ✗ | ✓ |
| on_configuration_change | Behaviour for streaming tables | ✗ | ✗ | ✓ |
