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

* `VARCHAR`, `STRING`, `TEXT` : default 2^24 size. max 2^27 size  
* `CHAR`, `CHARACTER`: same as `VARCHAR` but default size is 1
* `BINARY`, `VARBINARY`

### Boolean

Only for accounts 20260125+

### Date & time types

* `DATE`
* `TIME`
* `DATETIME`, `TIMESTAMP`, `TIMESTAMP_NTZ`: no time zone (provided time zone ignored)
* `TIMESTAMP_LTZ`: local time zone (provided time zone ignored)
* `TIMESTAMP_TZ` : with time zone

### Semi structured data types

* `VARIANT`
* `OBJECT`
* `ARRAY`

### Structured data types

* `ARRAY`
* `OBJECT`
* `MAP`

### Unstructured data type

* `FILE`

### Geospatial data types

* `GEOGRAPHY`
* `GEOMETRY`

### Vector data type

* `VECTOR`

## User Defined Functions

* extend built-in functions
* `CREATE OR REPLACE FUNCTION`
* must **return** value
* can be written in 
	* SQL
	* Java
	* Python
	* Scala
	* JavaScript
* if a query calls a UDF to access staged files,
	* and the query  SQL statement queries a view that calls any UDF/UDTF
	* it fails with an user error
* UDFS process files serially
* `CURRENT_SCHEMA` & `CURRENT_DATABASE` in UDF refers to schema/database of UDF not session
* if a query references staged files, that are modified/deleted during query run => error
* handler code can be inline or staged
* JavaScript & SQL must be inline


## User Defined Table functions

* UDFs that return a tabular value for each row
* can process files in parallel

## Stored Procedures

* `CREATE OR REPLACE PROCEDURE`

| Create Stored Procedure When 								| Create UDF when 											|
| --------------------------------------------------------	| ------------------------------------------------			|
| migrating Stored Procedure from another system 			| Migrating UDF from another system 						|
| administrative tasks (deleting old files, adding users) 	| callable function in SQL query that returns usable value 	|
|  															| get value for every input row/group						|
| DML statements (eg: `UPDATE`) 							| simple `SELECT` queries 									|


## Streams

* records tables' DML changes => CDC
* takes initial snapshot of source object's every row by initializing a point in time i.e. **offset** as the current transactional version of the object
* does **not store data**
* stores only offset & records further changes
* table schema changes


## Standard streams

* supported for 
	* standard tables
	* dynamic tables
	* Snowflake managed Apache Iceberg tables
	* directory tables
	* views
* tracks all DML changes
* performs join on inserted & deleted rows to provide row level delta
* cannot retrieve change data for geospatial data (use append-only streams instead)


## Append only streams

* supported for
	* standarad tables
	* dynamic tables
	* Snowflake managed Apache Iceberg tables
	* views
* tracks inserts only
* specifically returns inserted rows => performant for **ELT** (not ETL) & other such row insert reliant scenarios
* cannot create append only stream in target account using a secondary object as source 

## Insert only streams

* supported for 
	* external tables
	* externally managed Apache Iceberg tables
* overwritten or appended files are handled as new files

## Data Retention

* offset falls outside data retention period -> stream is *stale*
* stale => historical & unconsumed change records are inaccessible
* To continue tracking new changes, `CREATE STREAM` again.
* To prevent staleness
	* consume stream records within a DML statements during the table's retention period
	* regularly consume change data before `STALE_AFTER` timestamp
	* call `STREAM$STREAM_HAS_DATA` (it must return `FALSE` & stream must be empty)
* if retention period < 14 days and stream unconsumed, Snowflake temporarily extends max 14 (irrespective of edition)
	* configurable by `MAX_DATA_EXTENSION_TIME_IN_DAYS`
* check if stream is stale using `DESCRIBE STREAM` or `SHOW STREAMS`