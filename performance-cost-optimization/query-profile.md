# Query Profile

## Plans

* Query Execution Plans appear at the Query Profile's center
* Composed of 
	* operator nodes (represent rowset operators)
	* arrows (represent rowsets that flow from 1 operator to another)
* Operator Node includes:
	* Operator Type & ID
	* Execution time (% of total query time)
	* Detail preview (eg: table name or expression list)

* `EXPLAIN [ USING { TABULAR | JSON | TEXT } ] <statement>` returns *explain plan* not *actual plan*.


### Hybrid table usage



## Data Spilling

* When operation doesn't fit in memory, it *spills* to disk
* 1st to local disk, 2nd to remote disk
* bad for query performance
* Recommendations:
	* use larger warehouse
	* process data in smaller batches

```
	SELECT queryid, queryText, bytes_spilled_to_local_storage, bytes_spilled_to_remote_storage
	FROM snowflake.account_usage.query_history

```


## Data cache

1. Results Cache
	* expires after 24h
	* Hit conditions:
		* new query **exactly same** as old query w/o difference in syntax.
			* lowercase vs uppercase => cache not hit
			* aliasing => cache not hit
		* only reusable functions should be used
			* `UUID_STRING` => cache not hit
			* `RANDOM` => cache not hit
			* `RANDSTR` => cache not hit
		* no external functions
		* no select on *hybrid tables*.
		* table data contributing to query result remains unchanged
		* persisted result of previous query is still available
		* role accessing cached results has required privileges
			* `SELECT` query : 2nd query must have required access on tables used in query
			* `SHOW` query: 2nd query must have matching role as the 1st query
		* any configuration affecting the result must be unchanged
		* micro-partitions remain unchanged
	* Even if above conditions are met, cache use is still **not guaranteed**.
	* can be overriden at account, user & session level via `USE_CACHED_RESULT` session paramater
2. Local Disk Cache
	* data is retrieved from Remote Disk
	* stored in SSD & in memory
3. Remote Disk
	* long term storage
	* data *resilience* (AWS: 99.9999999%)


## Micro partition pruning

* micro-partitions enable precise column pruning at query run time
	* includes columns having semi-structured data
* query that accesses 10% range should access only 10% of micro-partitions
* columnar scanning of partitions => entire partition is not scanned if query specifies only 1 column
* sub-second response for time series of data for queries fine grained within 1hr or less
* no pruning for predicates with subqueries (even if subquery returns constant)


* all Snowflake table data is automatically divided into micro-partitions
* micro-partitions = contiguous units of storage
* 50 MB < micro-partition < 500 MB of **uncompressed** data
* *actual* size is smaller as Snowflake data is always **compressed**
* table row groups are mapped into micro-partitions, organized in a columnar fashion
* table can contain hundreds of millions of micropartitions
* metadata on all rows in a micropartition is stored, including :
	* range of values for each column
	* number of distinct values
	* additional properties used for optimization & efficient query processing


## Query History


* Snowsight
	* Query History page
	* Grouped Query History page
* ACCOUNT_USAGE schema
	* `QUERY_HISTORY` view
	* `AGGREGATE_QUERY_HISTORY` view
* INFORMATION_SCHEMA
	* `QUERY_HISTORY` functions

### Query History : Snowsight

* last 14 days
* steps
	1. Navigation menu
	2. Monitoring
	3. Query History
	4. Filter view
	5. Load more

### Grouped Query History : Snowsight

* based on `AGGREGATE_QUERY_HISTORY` view
* queries are grouped by **parametrized query hash ID**
* can drill down into individual queries
* particularly useful for
	* **Unistore workloads** : execute a small number of distinct statements at high throughput
	* Hybrid tables
* steps:
	1. Navigation menu
	2. Monitoring
	3. Query History
	4. Grouped Queries
* Privileges required. Any 1 of the following:
	* `ACCOUNTADMIN`
	* `IMPORTED PRIVILEGES` on the Snowflake database
	* `GOVERNANCE_VIEWER`

### Reasons for missing queries

* Still running
* Failed to run => no query profile
* not enough privileges
* > 14 days ago
* depth of job query detail & query profile metrics are *best effort*


## `Query_History` view

* present in `ACCOUNT_USAGE` & `READER_ACCOUNT_USAGE` schmeas
* `reader_account_name` & `reader_account_deleted_on` are unique to `READER_ACCOUNT_USAGE` schema
* 365 days history
* cancelled queries are identified by `error_message` rather than `execution_status`
* For `PUT` & `GET` commands, `execution_status` success => query authorized **not** files uploaded/downloaded
* `QUERY_ACCELERATION_ELIGIBLE` view in `ACCOUNT_USAGE` : identify eligible queries for **Query Acceleration Service**


## `QUERY_HISTORY` functions in `INFORMATION_SCHEMA` 

* `QUERY_HISTORY(END_TIME_RANGE_START, END_TIME_RANGE_END, RESULT_LIMIT, INCLUDE_CLIENT_GENERATED_STATEMENT)`
* `QUERY_HISTORY_BY_SESSION(SESSION_ID, END_TIME_RANGE_START, END_TIME_RANGE_END, RESULT_LIMIT, INCLUDE_CLIENT_GENERATED_STATEMENT)`
* `QUERY_HISTORY_BY_USER(USER_NAME, END_TIME_RANGE_START, END_TIME_RANGE_END, RESULT_LIMIT, INCLUDE_CLIENT_GENERATED_STATEMENT)`
* `QUERY_HISTORY_BY_WAREHOUSE(WAREHOUSE_NAME, END_TIME_RANGE_START, END_TIME_RANGE_END, RESULT_LIMIT, INCLUDE_CLIENT_GENERATED_STATEMENT)`

