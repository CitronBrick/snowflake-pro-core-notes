# Commands to unload data 

## GET
 
* internal stage -> client's local directory 
* internal stage
	* named internal stage
	* internal stage of table
	* internal stage of current user
* can be used after `COPY INTO`
* Unsupported operations:
	* downloading from external changes
	* downloading multiple files into divergent paths
	* cannot be executed from *Worksheets* in Web UI => use *SnowSQL* instead
	* renaming files
* In *Query History*, for `PUT` & `GET` commands `EXECUTION_STATUS` Success does not imply successful up/download only authorizatoin success
* `internal_stage` parameter
	* `@[<namespace>.]<int_stage_name>[/<path>]` : downloaded from named internal stage
	* `@[<namespace>.]%<table_name>[/path]` : downloaded from table stage
	* `@~[/<path>]` : downloaded from current user stage
* `PARALLEL` parameter

## LIST (LS)

* list items in 
	* stage (argument similar to `GET`)
		* internal named
		* specific table
		* current user
		* external named
	* Git repo cloned in Snowflake
		* `@[<namespace>.]<repository_clone>/branches/<branch_name>`
		* `@[<namespace>.]<repository_clone>/tags/<tag_name>`
		* `@[<namespace>.]<repository_clone>/commits/<commit_hash>`
* `PATTERN=<pattern>` can be used


## COPY INTO <table>

* load data from stage to table
	* named internal stage ( or user / table stage)
	* named external stage
	* external location
* `VALIDATION_MODE`
	* does **not load** data
	* validates data
	* not supported for Iceberg tables
	* `RETURN_<n>_ROWS` : validates `n` rows & fails at 1st error
	* `RETURN_ERRORS` : returns errors on files specified in `COPY INTO` command
	* `RETURN_ALL_ERRORS` : returns errors on files specified in `COPY INTO` command, including partially loaded files
		* partial loading happens due to loading with `ON ERROR` set to `CONTINUE`
* `DATE_FORMAT`
* `TIME_FORMAT`
* `TIMESTAMP_FORMAT`
* `ESCAPE`
* `TRIM_SPACE`
* `NULL_IF`
* `REPLACE_INVALID_CHARACTERS`
* `NULL_IF`
* zip options
* parse / skip header


## COPY INTO <location>

* unload from table/query into files in 
	* named internal stage
	* named external stage
	* external table
* `@[<namespace>.]<int_stage_name>[/path]`
* `@[<namespace>.]<ext_stage_name>[/path]`
* `@[<namespace>.]%<table_name>[/path]` 
* `@~[/<path>]` : user stage
* `VALIDATION_MODE` : `RETURN ROWS` (does **not copy** data, only validates)
* `DATE_FORMAT`
* `TIME_FORMAT`
* `TIMESTAMP_FORMAT`
* `ESCAPE`
* `TRIM_SPACE`
* `NULL_IF`
* `REPLACE_INVALID_CHARACTERS`
* `NULL_IF`
* zip options
* parse / skip header



## CREATE STAGE

* creates new named internal / external stage
* can include directory table

```
CREATE [ OR REPLACE ]  [ {TEMP | TEMPORARY}] STAGE [IF NOT EXISTS] <internal_stage_name>
	{ internalStageParams | externalStageParams }
	directoryTableParams
	[ FILEFORMAT = ( { FORMAT_NAME = '<file_format_name>' | TYPE = { CSV | JSON | AVRO | ORC | PARQUET | XML | CUSTOM } [formatTypeOptions] })]
	[ COMMENT = '<..>']
	[ [WITH] TAG ( <tag_name> = '<tag_value>') [, <tag_name> = '<tag_value>') * ] ]
```


```
CREATE OR ALTER [{TEMP | TEMPORARY}] STAGE <stage_name>
	{ internalStageParams | externalStageParams }
	directoryTableParams
	{ FILEFORMAT = ( { FORMAT_NAME = '<file_format_name>' TYPE = { CSV | JSON | AVRO | ORC | PARQUET | XML | CUSTOM } [ formatTypeOptions ]})}
	[ COMMENT = '<...>' ]
```

```
CREATE [OR REPLACE] STAGE  [ IF NOT EXISTS ] <name> CLONE <source_stage>
```

### Temporary stages

* external temporary stage dropped =>
	* stage itself is dropped
	* data files remain
* internal temporary stage dropped =>
	* data files deleted (irrespective of load status)
	* cannot be recovered (maintain copies outside)

### File format
	* loading data from stage via `COPY INTO <table>` : csv, json, avro, orc, parquet, xml, custom
	* unloading data to stage `COPY INTO <location>` : csv, json, parquet
	* default : csv

### internalStageParams 
	* Encryption
		* `SNOWFLAKE_FULL` (default)
			* Tri Secret Secure 
			* Client & server side encryption
			* Files are encrypted by client
			* Default: 128 bit key. 
			* Configure `CLIENT_ENCRYPTION_KEY_SIZE` to increase to 258 bit key
		* `SNOWFLAKE_SSE`
			* Server side encryption only
			* encrypted by cloud service
			* useful for querying pre-signed urls for cloud files


## CREATE FILE FORMAT


```
CREATE [OR REPLACE] [ { TEMP | TEMPORARY | VOLATILE}] FILE_FORMAT [ IF NOT EXISTS ] <name>
[ TYPE = { CSV | JSON | AVRO | ORC | PARQUET | XML | CUSTOM } [ formatTypeOptions ] ]
[ COMMENT = '' ]
```

* creates a named file format
* variant : `CREATE OR ALTER FILE FORMAT`
* options include:
	* record delimiter
	* row delimiter
	* date & time formats
	* skip header
	* skip blank lines
	* trim
	* empty field as null
	* null_if



