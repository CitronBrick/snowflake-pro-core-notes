# Snowflake Storage Concepts

## Micro partitions

* traditional (not Snowflake) : static partitioning  -> maintenance overhead & data skew
* micropartioning avoids limitations of static partitioning
* micropartitions = contiguous units of storage
* 50 mb < 1 micropartition uncompressed data < 500mb (data is always compressed)
* auto-enabled
* micropartition metadata
	* range of values
	* number of distinct values
	* additional optimization parameters 


### Benefits of Micropartitioning

* derived automatically, no setup
* extremely efficient
* overlaping value ranges + uniformly small size -> prevent skew
* columnar storage : columns are stored **independently within** micropartitions -> effective column scanning
* columns are compressed (Snowflake chooses algorithm) individually within micropartitions

### Impact of Micropartitioning

* deleting all rows -> deleting metadata
* dropping a column -> micropartitions are not rewritten
* pruning : query accessing 10% of values -> 10% of micropartitions are scanned
	* predicate with subquery -> no pruning even with constant subquery result
	* Time series response time < 1s

## Clustering

* In general, Snowflake produces well-clustered data in tables, which erodes due to DML over time = **Natural Clustering**
* Clustering is alternative to manually 
	1. sort by key columns
	2. reinsert
* A *clustered* table has a **defined** *clusetering key* (columns/expressions)
* Intended for
	* performance regardless of cost
	* performance outweighs cost
* Materialized views can be clustered
	* avoid clustering base tables, if possible
* Hybrid tables cannot be clustered (always sorted by primary key)
* Start clustering if:
	* query time has started degrading or is longer
	* **clustering depth** of table is large
* Reclustering is automatic (manual deprecated since 2020)
* After clustering key is defined, Snowflake **does not immediately** re-sort rows. It happens when needed automatically.

### Clustering depth

* For a populated table, measures the average depth (1+) of the overlapping micropartitions
* smaller depth ~ = better clustering
* not an absolute indicator of better clustering
* a table with no micropartitions (empty table) has clustering depth 0
* Eg: `SELECT SYSTEM$CLUSTERING_DEPTH('tpch_orders', '(C2, C9)', 'C2 = 25')`

### Benefits of defining clustering key for very large tables

* Improved scan efficiency (by skipping data not matching with predicates)
* better column compression. Especially true when other columns are strongly correlated with key column

### Selecting cluster keys : Strategies 

1. filters and `join` keys
2. `group by`, `order by` keys

* cardinality should be 
	* high enough for effective pruning
	* low enough to effectively group rows in same micropartition
* if cardinality is too high -> use column expression (eg: timestamp -> date, truncating numbers)
* specify clustering columns from **low to high** cardinality
* if text key is used for clustering -> Only 1st 5-6 bytes will be used


## Data Monitoring in Snowflake

### `STORAGE_USAGE` view


* `SNOWFLAKE.ACCOUNT_USAGE.STORAGE_USAGE` or `SNOWFLAKE.READER_ACCOUNT_USAGE.STORAGE_USAGE`
* displays average of daily data storage in bytes for last 365 days, across account
* includes both database & stage files
* Columns:
	* `USAGE_DATE` (local time)
	* `STORAGE_BYTES` : includes Time Travel bytes
	* `STAGE_BYTES` : bytes in internal storage
	* `FAILSAFE_BYTES`
	* `HYBRID_TABLE_STORAGE_BYTES`
	* `ARCHIVE_STORAGE_COOL_BYTES`
	* `ARCHIVE_STORAGE_COLD_BYTES` : includes active/fail-safe/Time Travel bytes
	* `ARCHIVE_STORAGE_RETRIEVE_TEMP_BYTES` : bytes used in standard storage tier during cold storage tier retrieval
* latency
	* <= 2 hours in `ACCOUNT_STORAGE`
	* <= 24 hours in `READER_ACCOUNT_USAGE`

### `BACKUP_STORAGE_USAGE` view

* stores information about backups (all Snowflake editions. Business Critical Edition includes retention lock & backup with legal hold)
* same table may be included in multiple table/schema/database backups
* Columns
	* `LOGICAL_BYTES`
	* `INCREMENTAL_BYTES_FROM_PREVIOUS_BACKUP` : nb of micropartition bytes present in backup but absent in previous backup
	* `DECREMENTAL_BYTES_FROM_PREVIOUS_BACKUP` : nb of micropartition bytes absent in backup but present in previous backup 
	* 0 is the value for above 2 columns for the oldest backup
* latency <= 6 hours

