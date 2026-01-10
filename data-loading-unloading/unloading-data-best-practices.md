# Best practices while unloading data

## File size & formats

* Location: local, S3, Google Cloud Storage, Azure
* Formats: Delimited (tsv, csv, etc.), JSON, parquet
* Encoding: UTF-8 only

### Compression


* gzip
* bzip2
* Brotli
* ZStandard

By default gzip compression is used, unless:
	* other method is specified or 
	* compression is explicitly disabled

## Empty Strings & NULL values

* Typically null values : 2 successive delimiters ,,
* Typically empty strings : ''
* `FIELD_OPTIONALLY_ENCLOSED_BY`
	* Default: `None`
	* When field already contains the specified character, escape using same character
		* eg: "A" enclosed by " => ""A"" 
* `EMPTY_FIELD_AS_NULL` : TRUE (default) or FALSE
* `NULL_IF` 
	* default: `\\N`
	* converts SQL null values to 1st value in the list

## Unload to single file

* In `COPY INTO` command use `SINGLE=TRUE` (default: `FALSE` )
* Increase `MAX_FILE_SIZE` to accomodate the large size
* potentially decreased performance
* `FILE_EXTENSION` is **ignored** & file is created **without extension**.
	* To avoid this, specify the filename & extension in the path
* If `COMPRESSION` is explicitly set, specified location must have the appropriate extension.

```
COPY INTO @mystage/myfile.csv.gz FROM mytable
FILE_FORMAT = (TYPE=csv COMPRESSION='gzip')
SINGLE = true
MAX_FILE_SIZE = 4900000000;
```

## Unload relationnal tables

### Unload relationnal tables to JSON

* Use `OBJECT_CONSTRUCT` function with `COPY` command to convert rows to single `VARIANT` column

```
COPY INTO @mystage
FROM  (SELECT OBJECT_CONSTRUCT('id', id, 'first_name', first_name, 'last_name', last_name FROM my_table ) 
FILE_FORMAT ( TYPE=json);
```

* The above command results in data_0_0_0.json.gz (gz is default compression)

### Unload relationnal tables to parquet data types

* Use `HEADER=TRUE` to include header
* By default
	* fixed point number => DECIMAL
	* floating point number => DOUBLE
* Use `CAST` to convert types
* Unloading floating point numbers to
	* CSV, JSON => truncated to ~ (15,9)
	* Parquet => not truncated

| Snowflake Logical  	| Parquet Physical 	| Parquet Logical	|
| ----------------		| -------------	 	| --------------	|
| TINYINT				| INT32				| INT(8)			|
| SMALLINT				| INT32				| INT(16)			|
| INT 					| INT32				| INT(32)			|
| BIGINT				| INT64				| INT(64)			|
| FLOAT 				| FLOAT 			| N/A 				|
| DOUBLE 				| DOUBLE 			| N/A 				|

```
COPY INTO @mystage
FROM ( 
	SELECT CAST(c1 as TINYINT), CAST(c2 as SMALLINT), CAST(c3 AS INT), CAST(c4 as BIGINT) 
	FROM mytable
)
FILE_FORMAT (type = PARQUET);
```


