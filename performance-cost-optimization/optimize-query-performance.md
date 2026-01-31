# Optimize Query Performance

## Materialized views

* *pre-computed* **stored** dataset derived from `SELECT` statement
* Snowflake EE+
* based on **1 table** only
* optimize query performance for
	* **common/repeated** query patterns returning **few columns/rows** compared to base table


## SPECIFIC SELECT COMMANDS

* choose only columns you need




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
