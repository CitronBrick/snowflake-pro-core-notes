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

