# Snowflake Storage Concepts

## Micro partitions

* traditional (not Snowflake) : static partitioning  -> maintenance overhead & data skew
* micropartioning avoids limitations of static partitioning
* micropartitions = contiguous units of storage
* 50 mb < 1 micropartition uncompressed data < 500mb (data is always compressed)
* auto-enabled
* micropartition metadata
	* range of values
	* number of distinct values
	* additional optimization parameters 


### Benefits of Micropartitioning

* derived automatically, no setup
* extremely effecient
* 