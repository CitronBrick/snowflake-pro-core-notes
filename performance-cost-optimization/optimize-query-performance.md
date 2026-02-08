# Optimize Query Performance


| Optimization  				| Storage cost 	| Compute cost 	|
|------------------------------	|------------ 	|-------------	|
| Search Optimization Service 	| Y 			| Y 			|
| Query Acceleration Service 	| N 			| Y 			|
| Materialized view 			| Y 			| Y 			|
| Clustering the table 			| Y 			| Y 			|

* The storage cost of table clustering is due to rewriting existing partitions to new partitions (no new rows added)

## Materialized views

* *pre-computed* **stored** dataset derived from `SELECT` statement
* Snowflake EE+
* based on **1 table** only
* optimize query performance for
	* **common/repeated** query patterns returning **few columns/rows** compared to base table
	* equality searches
	* range searches
	* sort operations


## SPECIFIC SELECT COMMANDS

* choose only columns you need


## Table clustering

* Snowflake supports automatic clustering based on key (1 or more columns)
* tables & materialized views can be clustered (**not hybrid tables**)
* in `CREATE TABLE` or `ALTER TABLE` statement end with `CLUSTER BY`
	* `CLUSTER BY (c1, c2)`
	* `CLUSTER BY (to_date(c1), substring(c2, 0 , 0))`
	* `CREATE TABLE T3 ( t timestamp, v variant) cluster by (v:"Data":id::number)`
* **Clustering depth** : average depth (1+) of the overlapping micropartitions for specified columns. (0 for empty table)
* Clustering depth is **not** an absolute way to check if table is well clustered. Query performance is.
* System functions : `SYSTEM$CLUSTERING_DEPTH`, `SYSTEM$CLUSTERING_INFORMATION`
* To drop: `ALTER TABLE t2 DROP CLUSTERING KEY`

### Clustering key

* max 3-4 columns recommended
* use columns that are most used in selective filters
* if more columns needed, add columns used in join predicates
* **Cardinality** : nb of distinct values
* Cardinality ideally should be:
	* **high enough** for effective pruning
	* **low enough** to effectively group rows in the same micro-partitions
* if cardinality too high, maintenance cost > benefits (especially if point lookups are not used much)
* if we want to use a clustering key with cardinality too high,
	* reduce cardinality by using a column expression instead of the direct value
	* eg: if column is timestamp, cast to date in clustering key
	* if column is number, trucate it eg: `TRUNC(234532581, -5)`
* if clustering column is text, Snowflake reduces to 1st 5 bytes
* Order clustering columns from **lowest to highest** cardinality
* if filter & `join` keys are different from `group by` & `order by` keys, prefer the former.
* post DML operations, automatic reclustering is done
* reclustering -> new micropartitions created -> old micropartiions remain for TimeTravel & Fail-safe retition periods -> storage costs


## SEARCH OPTIMIZATION SERVICE

* Snowflake EE+
* improves performance of certain analytical & lookup queries
	* point lookup queries => returns small nb of distinct rows
	* Character Data & IP addresss searches using `SEARCH` & `SEARCH_IP` FUNCTIONS
	* `LIKE`, `ILIKE`, `RLIKE`
	* Queries on elements in `VARIANT`,`OBJECT` & `ARRAY` (semi-structured) columns using the following predicates
		* equality
		* `IN`
		* using `ARRAY_CONTAINS`
		* using `ARRAYS_OVERLAPS`
		* using `SEARCH`
		* substring & regexp predicates
		* `NULL` check
	* Queries on elements in structured `ARRAY`, `OBJECT`, `MAP` (structured) columns using the following predicates
		* equality
		* `IN`
		* substring
	* Geospatial functions 
* `ALTER TABLE my_table ADD SEARCH OPTIMIZATION`

## Persisted Query Results

* query results are cached for 24 hours
* after query reuse, cached again for 24 hours (max 31 days since 1st query)
* Security token
	* Token provided by Snowflake Connector for Spark expires after 24 hr irrespective of cached results size
	* access token for large results (> 100 kb) expires after 6 hr
	* smaller results need no access token
* if query is reused, no execution
* Cache not hit if 
	* queries have differences in uppercase/lowercase
	* queries have different table aliases
	* non-reusable functions are used : `UUID_STRING`, `RANDOM`, `RANDSTR`
	* external functions are used
	* **hybrid tables** are queried
	* table data contributing to result has changed
	* configuration options (affecting result production) have changed
	* micro partitions have changed
* Even if all above conditions are met, cache hit is still **not guaranteed**
* Role to access cached results need below privileges:
	* `SELECT` : role must have necessary access across all tables queried
	* `SHOW` : role must match role that generated the cache results
* enabled by default
* override by using `USED_CACHE_RESULT` paramter at  *account*, *user*, *session* level

### Post-processing of query results

* Needed when eg:
	* you need to run q2 on top of pre-computed q1
	* q1 is `SHOW`, `DESCRIBE`, `CALL` with a result that's not easy to reuse
* Needed for stored procedure (not function !) inside a more complex SQL statement
* Solutions : `RESULT_SCAN` or pipe operator `-->`
* Eg: show tables that have 0 rows

```

SHOW TABLES;

SELECT "schema_name", "name" as "table_name"
FROM table(RESULT_SCAN(LAST_QUERY_ID()))
WHERE rows = 0;
```

* Pipe operator avoids showing q1 results

```
SHOW TABLES
	--> SELECT "schema_name", "name" as "table_name" 
		FROM $1
		WHERE rows = 0;
```


### Impact of different caching types

##### Results Cache = see Persisted Query Results

#### Warehouse cache

* running warehouse maintains cache of table data
* accessible for queries running on the same warehouse


```
SELECT warehouse_name, count(*), sum(percentage_scanned_from_cache)
FROM snowflake.account_usage.query_history
WHERE start_time >= dateadd(month, -1, current_timestamp()) and bytes_scanned > 0
GROUP BY 1 ORDER BY 3;
```

##### Auto-suspension of warehouses


* cache deleted when warehouse is suspended 
* Auto-suspension not recommended when running frequent small queries
* Auto-suspension time limit guidelines
	* tasks => immediate suspension
	* DevOps, DataOps, Data Science => cache unimportant => 5min
	* BI & `Select` => 10min+
* a running warehouse consumes credits even while sitting idle

#### Metadata cache

* Does not use warehouse
* no time limit
* used when `count`, `min`, `max` are run
* not affected by table data changes
* improves compile time for queries on commonly used tables
* cf [https://snowflake.discourse.group/t/what-is-the-difference-between-metadata-cache-and-result-cache/2419/3](https://snowflake.discourse.group/t/what-is-the-difference-between-metadata-cache-and-result-cache/2419/3)

