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