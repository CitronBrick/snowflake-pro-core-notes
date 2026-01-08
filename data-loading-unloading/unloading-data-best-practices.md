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

* By default null values 