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


