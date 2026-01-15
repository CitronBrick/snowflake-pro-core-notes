# Snowflake catalog & objects

## Databases

* can be cloned
* data retention time
* data extension time
* external volume
* catalog
* replace invalid characters
* default ddl collation
* storage serlialization policy
* comment
* catalog sync
* catalog sync namespace mode: nested, flatten
* catalog sync namespace flatten delimiter
* with contact
* object visibility
* data compaction
*  name : unique within account
* `CREATE OR ALTER DATABASE`
* `CREATE OR REPLACE DATABASE`
* `CREATE DATABASE ... CLONE`
* `CREATE DATABASE FROM BACKUP LIST`

## Stages

* temporary area to store data to be loaded to tables
* types
	* external
	* internal 
		* named
		* user
		* table

## Schema types

* TRANSIENT
	* contains only transient tables
* PERMANENT (default)
	
## Table types


| Type 			 | Persistence  					| Cloning Target   					| Time Travel (days)  			| Fail-safe Period (days) 	|
|--------------	 | -----------------------------	|---------------------------		| -----------------------------	|-------------------------	|
| Temporary 	 | Session end 						| Temporary, Transient 				| 0 or 1 (default)				| 0							|
| Transient 	 | Until explicitly dropped 		| Temporary, Transient 				| 0 or 1 (default)				| 0							|
| Permanent (SE) | Until explicitly dropped 		| Temporary, Transient, Permanent 	| 0 or 1 (default) 				| 7 						|
| Permanent (EE+)| Until explicitly dropped 		| Temporary, Transient, Permanent 	| 0 - 90 						| 7 						|


## View types 

* Materialized : query expression whose results are stored
* Non-materialized: query expression which is recalculated each tiem it's referenced
	* Recursive : refers to itself
	* Temporary : `TEMP`, `TEMPORARY`, `VOLATILE` : dropped at session end
* Secure views
	* can be Materialized or Non-materialized
	* does not expose query to users
	* `CREATE OR REPLACE`

## Data types

### Numeric Data Types

* `NUMBER`, `DECIMAL`, `NUMERIC` :  (38,0)
* `INT`, `INTEGER`, `BIGINT`, `SMALLINT`, `TINYINT`, `BYTEINT` : synonymous with `NUMBER` but no precision  / scale
* `FLOAT`, `FLOAT4`, `FLOAT8`, `DOUBLE`, `DOUBLE PRECISION`, `REAL`
* `DECFLOAT` : 38 significant digits of precision & dynamic base 10 exponent

### String & Binary types

* `VARCHAR` : default 2^24 size. max 2^27 size  
