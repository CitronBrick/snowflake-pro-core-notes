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


### Standard streams

* supported for 
	* standard tables
	* dynamic tables
	* Snowflake managed Apache Iceberg tables
	* directory tables
	* views
* tracks all DML changes
* performs join on inserted & deleted rows to provide row level delta
* cannot retrieve change data for geospatial data (use append-only streams instead)


### Append only streams

* supported for
	* standarad tables
	* dynamic tables
	* Snowflake managed Apache Iceberg tables
	* views
* tracks inserts only
* specifically returns inserted rows => performant for **ELT** (not ETL) & other such row insert reliant scenarios
* cannot create append only stream in target account using a secondary object as source 

### Insert only streams

* supported for 
	* external tables
	* externally managed Apache Iceberg tables
* overwritten or appended files are handled as new files

### Streams on Views

* supported for local & Snowflake Secured Data Sharing views & secure views
* cannot track changes on materialized views
* change tracking must be enabled in the underlying table

#### Stream on View on Join

* contains : delta(t1) x t2  + t1 x delta(t2) + delta(t1) x delta(t2)
* above compute cost is not always linear


### Data Retention

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


### Multiple Consumers

* A stream stores only offset **not actual data**
* The offset moves only when DML runs (including `CTAS` or `COPY INTO <location>`)
* If multiple consumers (tasks, script etc.) consume the same stream, they cannot correctly capture their own change data,
  since each of them move the offset
* Recommendation: Create separate stream for each consumer. No significant cost


### Changes clause

* alternative to Streams
* Read only
* does not advance the offset
* needs `AT`/`BEFORE` , `ENDS` (optional) clause
* adds several hidden metadata columns

### Stream limitations


* not supported for external partitioned tables
* not supported for Apache Iceberg tables with external catalog
* cannot track changes on views with `GROUP BY`
* When task is triggered by Stream on View, it will be also triggered during any changes to underlying tables
* modifying nullability on a column, might cause stram query failure due to impermissible `NULL` values. The stream will enforce current nullability constraint. 

## Tasks

* can be scheduled
* can be triggered by event
* can run SQL & Stored Proc.
* sequence of parallel/serial tasks = Task Graphs
* by default, runs as a **system service** *decoupled* from the user

1. Create task admin role

```

USE ROLE securityadmin;
CREATE ROLE taskadmin;

USE ROLE accountadmin;
GRANT EXECUTE TASK, EXECUTE MANAGED TASK ON ACCOUNT TO ROLE taskadmin;

USE ROLE securityadmin;
GRANT ROLE taskadmin to ROLE myrole;
```

Owner role of task is deleted -> delter becomes new owner & task is paused.

2. Create Task

```
CREATE [ OR REPLACE ] TASK [ IF NOT EXISTS ] <name>
	[ WITH TAG ( <tag_name> = <tag_value)+ ] 
	[ WITH CONTACT ( <purpose> = <contact_name>)+ ]
	[ ( WAREHOUSE = <string> ) | ( USER_TASK_MANAGED_INITIAL_WAREHOUSE_SIZE = <string> ) ]
	[ SCHEDULE = { <num>  ( HOURS | MINUTES | SECONDS ) } | USING CRON <expr> <time_zone> ]
	[ CONFIG = <configuration_string> ]
	[ OVERLAP_POLICY = { NO_OVERLAP | ALLOW_CHILD_OVERLAP | ALLOW_ALL_OVERLAP }] 
	[ (<session_parameter> = <value>)+ ]
	[ USER_TASK_TIMEOUT = <ms> ]
	[ SUSPEND_TASK_AFTER_NUM_FAILURES = <num> ]
	[ ERROR_INTEGRATION = <integration_name> ]
	[ SUCCESS_INTEGRATION = <integration_name> ]
	[ LOG_LEVEL = <level> ]
	[ COMMENT = <string> ]
	[ FINALIZE = <string> ]
	[ TASK_AUTO_RETRY_ATTEMPTS = <num> ]
	[ USER_TARGET_MINIMUM_TRIGGER_INTERVAL_IN_SECONDS = <num> ]
	[ TARGET_COMPLETION_INTERVAL= '<num> ( HOURS | MINUTES | SECONDS )']
	[ SERVERLESS_TASK_MIN_STATEMENT_SIZE = ( XSMALL | SMALL | MEDIUM | LARGE | XLARGE | XXLARGE ) ]
	[ AFTER <string>+ ]
	[ EXECUTE_AS_USER <user_name> ]
	[ WHEN = <boolean_expr> ]
AS <sql>
```