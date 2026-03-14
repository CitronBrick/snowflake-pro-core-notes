# Commands to load data

## Create Stage

Internal
```
CREATE OR REPLACE TEMP STAGE IF NOT EXISTS internal_stage_1
ENCRYPTION = TYPE = 'SNOWFLAKE_FULL'
DIRECTORY = ENABLE = TRUE
	REFRESH_ON_CREATE = TRUE
	AUTO_REFRESH = TRUE
FILE_FORMAT = TYPE = 'CSV'
COMMENT = 'my comment'
WITH TAG myTag;
```

External
```
CREATE OR REPLACE TEMP STAGE IF NOT EXISTS external_stage_1
URL = 's3://...'
AWS_ACCESS_POINT_ARN=''
ENCRYPTION = TYPE = 'AWS_SSE_S3'
DIRECTORY = ENABLE = TRUE
	REFRESH_ON_CREATE = TRUE
	AUTO_REFRESH = TRUE
FILE_FORMAT = TYPE = 'CSV'
COMMENT = 'my comment'
WITH TAG myTag;
```

```
Create or replace stage if not exists myClonedStage CLONE <int_or_ext_source_stage_name>
```

## CREATE File format

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


## CREATE PIPE

creates new pipe for defining the `COPY INTO <table>` statement used :
	* by Snowpipe to load from ingestion queue
	* or by Snowpipe streaming with high performance architecture to load directly from streaming source -> table

```
CREATE [OR REPLACE]  PIPE [IF NOT EXISTS] <name>
	[AUTO_INGEST= [TRUE|FALSE]]
	[ERROR_INTEGRATION=<integration_name>]
	
	[AWS_SNS_TOPIC='<topic>']
	[INTEGRATION=<integration_name>]
	[COMMENT = 'text']
	AS <copy statement>
```

## CREATE EXTERNAL TABLE

```
CREATE [OR REPLACE] EXTERNAL TABLE [IF NOT EXISTS] <table_name>
	(
		[ <col_name> <col_type> AS <expr> | <part_col_name> <part_col_type> AS <expr> ]
		[ inlineConstraint ]
		[ <col_name> <col_type> AS <expr> | <part_col_name> <part_col_type> AS <expr> ]
	)
	cloudProviderParams
	[ PARTITION BY ( <part_col_name> [ , <part_col_name> ] ) ]
	[ WITH ] LOCATION = externalStage
	[ REFRESH_ON_CREATE  = { TRUE | FALSE }]
	[ AUTO_REFRESH  = { TRUE | FALSE }]
	[ PATTERN = '<regex_pattern>' ]
	FILE_FORMAT = ( { FORMAT_NAME = '<file_format_name>' | TYPE = { CSV | JSON | AVRO | ORC | PARQUET } [ formatTypeOptions ] } )
	[ AWS_SNS_TOPIC = '<string>' ]
	[ COPY GRANTS ]
	[ COMMENT = '<string_literal>' ]
	[ [ WITH ] ROW ACCESS POLICY <policy_name> ON (VALUE) ]
	[ [ WITH ] TAG ( <tag_name> = '<tag_value>' [ , <tag_name> = '<tag_value>' , ... ] ) ]
	[ WITH CONTACT ( <purpose> = <contact_name> [ , <purpose> = <contact_name> ... ] ) ]

```



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

## INSERT / INSERT OVERWRITE

```
INSERT [ OVERWRITE ] INTO <target_table> [ (<target_col_name> [ , ...])]
	{
		VALUES ( { <value> | DEFAULT | NULL } [ ,... ] ) [ , (...) ]
		| query
	}
```

`OVERWRITE` = truncate & load (needs role with `DELETE` prvilege )

* limited to 200,000 VALUES

## PUT

uploads 1+ data files from local -> internal stage

```
PUT file://<absolute_path_to_file>/<filename> internalStage
	[ PARALLEL = <integer> ] 
	[ AUTO_COMPRESS = TRUE | FALSE ]
	[ SOURCE_COMPRESSION = AUTO_DETECT | GZIP | BZ2 | BROTLI | ZSTD | DEFLATE | RAW_DEFLAT | NONE ]
	[ OVERWRITE = TRUE | FALSE ]
```


## VALIDATE 

validates the files loaded in a past location of the `COPY INTO <table>` 
& returns (not only 1st) **all** the load errors

```
VALIDATE ( [<namespace>.]<table_name> , JOB_ID => { '<query_id>' | '_last' })
```

* query_id can be got from **Query History** page