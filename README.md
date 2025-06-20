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



____________________________________________________________________________________________


h2. Model-level Configs – Spark / Glue / Databricks

|| Config Key || Config Type || Adapter || Found in Docs || Found in GitHub || Notes ||
| materialized | model (general) | Spark | Yes | Yes | Core materialization (table/view/incremental/ephemeral). |
| materialized | model (general) | Glue | Yes | Yes | Inherits core behaviour. |
| materialized | model (general) | Databricks | Yes | Yes | Same as Spark. |
| schema | model (general) | Spark | Yes | Yes | Maps to Hive DB. |
| schema | model (general) | Glue | Yes | Yes | Glue database. |
| schema | model (general) | Databricks | Yes | Yes | Unity-Catalog schema. |
| database | model (general) | Spark | Yes | Yes | Alias of schema. |
| database | model (general) | Glue | – | No | Not used. |
| database (catalog) | model (general) | Databricks | (profile level) | Yes | Set via profile `catalog`. |
| alias | model (general) | Spark | Yes | Yes | Custom relation name. |
| alias | model (general) | Glue | Yes | Yes | – |
| alias | model (general) | Databricks | Yes | Yes | – |
| enabled | model (general) | Spark | Yes | Yes | Toggle resource. |
| enabled | model (general) | Glue | Yes | Yes | – |
| enabled | model (general) | Databricks | Yes | Yes | – |
| tags | model (general) | Spark | Yes | Yes | List of tags. |
| tags | model (general) | Glue | Yes | Yes | – |
| tags | model (general) | Databricks | Yes | Yes | – |
| meta | model (general) | Spark | Yes | Yes | Arbitrary key/val. |
| meta | model (general) | Glue | Yes | Yes | – |
| meta | model (general) | Databricks | Yes | Yes | – |
| persist_docs | model (general) | Spark | Yes | Yes | Table & col comments. |
| persist_docs | model (general) | Glue | No | No | Not implemented. |
| persist_docs | model (general) | Databricks | Yes | Yes | Needs Delta for col comments. |
| pre-hook / post-hook | model | Spark | Yes | Yes | SQL hooks. |
| pre-hook / post-hook | model | Glue | Yes | Yes | – |
| pre-hook / post-hook | model | Databricks | Yes | Yes | – |
| grants | model | Spark | Yes | No | Not supported (no grant engine). |
| grants | model | Glue | Yes | No | Same limitation. |
| grants | model | Databricks | Yes | Partial | Unity-Catalog WIP. |
| unique_key | incremental | Spark | Yes | Yes | Needed for merge. |
| unique_key | incremental | Glue | Yes | Yes | Hudi merge only. |
| unique_key | incremental | Databricks | Yes | Yes | Delta merge. |
| incremental_strategy | incremental | Spark | Yes | Yes | append / insert_overwrite / merge / microbatch. |
| incremental_strategy | incremental | Glue | Yes | Yes | append / insert_overwrite / merge(Hudi). |
| incremental_strategy | incremental | Databricks | Yes | Yes | append / insert_overwrite / merge / replace_where. |
| on_schema_change | incremental | Spark | Yes | Yes | ignore / fail / append_new_columns / sync_all_columns. |
| on_schema_change | incremental | Glue | Yes | Yes | Supported v1.9+. |
| on_schema_change | incremental | Databricks | Yes | Yes | – |
| file_format | table/incremental | Spark | Yes | Yes | parquet (def), delta, iceberg, hudi … |
| file_format | table/incremental | Glue | *Implicit* | Yes | Use hudi for merge. |
| file_format | table/incremental | Databricks | Yes | Yes | Default delta. |
| location_root | table | Spark | Yes | Yes | External path. |
| location_root | table | Glue | – | No | Use `custom_location`. |
| location_root | table | Databricks | – | Yes | Inherited; not documented. |
| custom_location | table | Glue | Yes | Yes | S3 override. |
| partition_by | table/inc | Spark | Yes | Yes | List of cols. |
| partition_by | table/inc | Glue | Yes | Yes | – |
| partition_by | table/inc | Databricks | Yes | Yes | – |
| cluster_by / buckets | table | Spark | Yes | Yes | Bucketing. |
| cluster_by / buckets | table | Glue | No | Yes | Works, undocumented. |
| cluster_by / buckets | table | Databricks | No | Yes | Works; prefer Z-order. |
| tblproperties | table | Spark | Yes | Yes | Key-value props. |
| tblproperties | table | Glue | Implicit | Yes | Pass-through. |
| tblproperties | table | Databricks | Yes | Yes | Delta props. |
| databricks_compute | model | Databricks | Yes | Yes | Select alt compute. |
| event_time | microbatch | Spark | Yes | Yes | Required for microbatch. |
| event_time | microbatch | Glue | – | No | Microbatch not supported. |
| event_time | microbatch | Databricks | Yes | Yes | – |
| batch_size / lookback / begin | microbatch | Spark | Yes | Yes | Microbatch keys. |
| batch_size / lookback / begin | microbatch | Databricks | Yes | Yes | – |
| incremental_predicates | incremental | Databricks | Yes | Yes | replace_where strategy. |
| full_refresh | model | Spark | Yes | Yes | Override CLI flag. |
| full_refresh | model | Glue | Yes | Yes | – |
| full_refresh | model | Databricks | Yes | Yes | – |


h2. Seed Configs

|| Config Key || Adapter || Found in Docs || Found in GitHub || Notes ||
| quote_columns | Spark | Yes | Yes | Quote CSV seed column names. |
| quote_columns | Glue | Yes | Yes | – |
| quote_columns | Databricks | Yes | Yes | – |
| column_types | Spark | Yes | Yes | Override data types. |
| column_types | Glue | Yes | Yes | – |
| column_types | Databricks | Yes | Yes | – |
| delimiter | Spark | Yes | Yes | CSV delimiter. |
| delimiter | Glue | Yes | Yes | – |
| delimiter | Databricks | Yes | Yes | – |
| seed_format | Glue | Yes | Yes | csv / parquet / json. |
| seed_mode | Glue | Yes | Yes | overwrite / append. |


h2. Snapshot Configs

|| Config Key || Adapter || Found in Docs || Found in GitHub || Notes ||
| strategy (timestamp / check) | Spark | Yes | Yes | Core strategies. |
| strategy | Glue | Yes | Yes | – |
| strategy | Databricks | Yes | Yes | – |
| updated_at | Spark | Yes | Yes | For timestamp strategy. |
| updated_at | Glue | Yes | Yes | – |
| updated_at | Databricks | Yes | Yes | – |
| unique_key | Spark | Yes | Yes | PK for snapshot. |
| check_cols | Spark | Yes (legacy) | Yes | Deprecated. |
| hard_deletes | Spark | Yes | Yes | v1.9+. |
| invalidate_hard_deletes | Spark | Yes | Yes | – |
| dbt_valid_to_current | Spark | Yes | Yes | v1.9+. |
| snapshot_meta_column_names | Spark | Yes | Yes | v1.9+. |
| snapshot_name | Spark | Yes | Yes | Custom table name. |
| target_schema | Spark | Yes | Yes | Override schema. |
| target_database | Spark | Yes | Yes | DB override (N/A for Glue). |



h2. Source Configs

|| Config Key || Adapter || Found in Docs || Found in GitHub || Notes ||
| quoting | Spark | Yes | Yes | Control quoting. |
| quoting | Glue | Yes | Yes | – |
| quoting | Databricks | Yes | Yes | – |
| loaded_at_field | Spark | Yes | Yes | Freshness check. |
| freshness (warn_after / error_after) | Spark | Yes | Yes | – |
| filter (freshness) | Spark | Yes | Yes | Optional filter. |
| overrides | Spark | Yes | Yes | Source overrides. |
| external | Spark | Yes | Yes | External tables (via packages). |


h2. Test Configs

|| Config Key || Adapter || Found in Docs || Found in GitHub || Notes ||
| severity | all | Yes | Yes | error / warn. |
| warn_if | all | Yes | Yes | Threshold expr. |
| error_if | all | Yes | Yes | Threshold expr. |
| fail_calc | all | Yes | Yes | Failure calc. |
| limit | all | Yes | Yes | Row limit. |
| store_failures | all | Yes | Yes | Save failures table. |
| store_failures_as | all | Yes | Yes | Custom fail table. |
| where | all | Yes | Yes | Extra filter. |


h2. Profile (Connection) Configs – Spark

|| Profile Key || Found in Docs || Found in GitHub || Notes ||
| type (spark) | Yes | – | Selects adapter. |
| method (session / thrift / http / odbc) | Yes | Yes | Connection transport. |
| driver (odbc) | Yes | Yes | Path to driver. |
| host | Yes | Yes | Thrift/HTTP host. |
| port | Yes | Yes | Default 10001/443. |
| user | Yes | Yes | Optional username. |
| token | Yes | Yes | Databricks PAT. |
| organization | Yes | Yes | Azure DBX only. |
| cluster / endpoint | Yes | Yes | Databricks cluster / SQL WH. |
| auth / kerberos_service_name | Yes | Yes | Thrift auth. |
| use_ssl | Yes | Yes | Thrift SSL flag. |
| connect_timeout / connect_retries | Yes | Yes | HTTP retry knobs. |
| server_side_parameters | Yes | Yes | Spark conf overrides. |


h2. Profile (Connection) Configs – Glue

|| Profile Key || Found in Docs || Found in GitHub || Notes ||
| type (glue) | Yes | – | Select adapter. |
| role_arn | Yes | Yes | IAM role for session. |
| region | Yes | Yes | AWS region. |
| schema | Yes | Yes | Glue database. |
| workers | Yes | Yes | Worker count. |
| worker_type | Yes | Yes | G.1X / G.2X. |
| idle_timeout | Yes | Yes | Idle mins. |
| session_provisioning_timeout_in_seconds | Yes | Yes | Session startup timeout. |
| query_timeout_in_minutes | Yes | Yes | Query timeout. |
| glue_version | Yes | Yes | 3.0 / 4.0. |
| security_configuration | Yes | Yes | Glue encryption config. |
| connections | Yes | Yes | Glue connections list. |
| conf | Yes | Yes | Spark --conf overrides. |
| extra_py_files | Yes | Yes | Extra Python libs. |
| extra_jars | Yes | Yes | Extra JARs. |
| delta_athena_prefix | Yes | Yes | Athena mirror tables. |
| tags | Yes | Yes | AWS resource tags. |
| location | Yes | Yes | Warehouse S3 path. |
| query-comment | Yes | Yes | Prepend SQL comment. |
| project_name | Yes | Yes | Must match dbt project. |


h2. Profile (Connection) Configs – Databricks

|| Profile Key || Found in Docs || Found in GitHub || Notes ||
| type (databricks) | Yes | – | Select adapter. |
| catalog | Yes | Yes | Unity-Catalog name (opt). |
| schema | Yes | Yes | Required schema. |
| host | Yes | Yes | Workspace host. |
| http_path | Yes | Yes | SQL WH / cluster endpoint. |
| token | Yes | Yes | PAT (default auth). |
| auth_type | Yes | Yes | `oauth` or omit for PAT. |
| client_id / client_secret | Yes | Yes | OAuth creds. |
| threads | Yes | Yes | Parallel threads. |
| connect_retries / connect_timeout | Yes | Yes | Connection retries. |
| compute (profile block) | Yes | Yes | Named alt compute targets. |




