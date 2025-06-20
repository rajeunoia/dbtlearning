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


h2. 1 – profiles.yml (connection-time) configs

|| Topic || dbt-Spark || dbt-Glue || dbt-Databricks || Why you care ||
| Connection method | `session`, `thrift`, `http`, `odbc` | `type: glue` (interactive Spark session over AWS API) | `type: databricks` (`http_path` to SQL Warehouse / cluster) | Pick the right transport & auth for each engine |
| Retry knobs | `retry_all`, `connect_timeout`, `connect_retries` | *Not implemented* – Glue sessions auto-retry | Same knobs supported | Tune flaky-network resilience |
| Session / cluster parameters | `server_side_parameters` (per-run Spark conf) | Glue-only extras (`workers`, `worker_type`, `glue_version`, etc.) | Databricks-only `compute:` block & `session_properties` | Engine-specific job / cluster settings |

h2. 2 – Seed-level configs (seeds: block)

|| Key || Spark || Glue || Databricks || Notes ||
| file_format | Any Spark source (`csv`, `parquet`, `delta`, …) | Same list | Typically `delta` or `parquet` | Delta seeds require Delta packages |
| location_root / custom_location | `location_root` path | **`custom_location`** instead | `location_root` works | Directory setting is adapter-specific |
| seed_format / seed_mode | n/a | Glue shorthands (`seed_format: csv`, `seed_mode: append`) | n/a | Bulk-load CSVs to S3 with Glue |
| CSV options (`header`, `delimiter`, `quote`, …) | ✓ | ✓ | ✓ | Behaviour identical across adapters |

h2. 3 – Snapshot configs (snapshots/...)

|| Gotcha || Spark || Glue || Databricks ||
| Supported file formats | `delta`, `hudi`, `iceberg` | **Only `hudi`** | `delta` *or* `hudi` |
| Strategies (`timestamp`, `check`) | ✓ | ✓ | ✓ |
| Large-table considerations | Standard Delta/Iceberg tuning | Choose CoW vs MoR Hudi table-type | Delta snapshots scale well; Unity Catalog supports 3-part names |

h2. 4 – Incremental *microbatch* configs

|| Microbatch field || Spark || Glue || Databricks ||
| `incremental_strategy: microbatch` | ✓ | ✗ (not yet) | ✓ |
| Other keys (`event_time`, `batch_size`, `begin`, `lookback`, `concurrent_batches`) | Honoured | Ignored | Honoured |

h2. 5 – Python-model / compute-selection (Databricks only)

|| Databricks-specific key || Purpose ||
| `submission_method` (`all_purpose_cluster` \| `job_cluster`) | Select cluster launch mode |
| `cluster_id` / `job_cluster_config` | Point to an existing cluster or define an ephemeral jobs cluster |
| `create_notebook` (`true` \| `false`) | Emit compiled PySpark as a Databricks notebook |
| `databricks_compute` | Named SQL Warehouse / compute resource for the run |


h2. 6 – Test & documentation edge-cases

|| Edge case || Spark || Glue || Databricks || Notes ||
| `store_failures` file format | Delta table | Parquet (unless `file_format` overridden) | Delta table | Controls how failing-row tables are materialised |
| Column comments via `persist_docs` | Works on Delta & Iceberg | Hive/Parquet metastore can lose column comments | Works on Delta (and Unity Catalog) | Column-level docs portability differs |

