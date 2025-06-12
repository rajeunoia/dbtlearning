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
