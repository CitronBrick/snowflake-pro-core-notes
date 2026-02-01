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
* **Clustering depth** : average depth (1+) of the overlapping micropartitions for specified columns. (0 for empty table)
* Clustering depth is **not** an absolute way to check if table is well clustered. Query performance is.
* System functions : `SYSTEM$CLUSTERING_DEPTH`, `SYSTEM$CLUSTERING_INFORMATION`

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
