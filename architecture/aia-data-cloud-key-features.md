# Snowflake AIA Data Cloud - Key features

## Interoperable storage

* Snowflake tables
* Apache Iceberg tables
* Hybrid tables


### Snowflake tables

* internal
* **compressed** columnar format
* structured, semi-structed, unstructured (`FILE` data type) data

### Apache Iceberg tables

* external
* S3, GCS, Azure
* performance of native Snowflake tables
* structured & semi-structed data

### Hybrid tables

* low latency, high throughput optimization
* index based random read & writes
* store structured & semi-structed data
* queries do not use **results cache**
* < 2TB
* cannot be temporary or transient

## Elastic Compute

### Virtual Warehouse

* cluster of compute resources
* process SQL statements
* run Scala, Java & Python code via **Snowpark**
* run Apache Spark via **Snowpark Connect for Spark**
* each Virtual Warehouse is *independent* => does not share compute with other Virtual Warehouse
* Multiple sizes : XSMALL - 6XLARGE
* Cache (dropped when Warehouse is *suspended*)
* Multi Warehouse Cluster
	* Maximized: min cluster = max cluster
	* Auto Scale : min cluster < max cluster

## Snowflake layers

> Database Storage -> Compute -> Cloud services

## Snowflake Editions

* Standard Edition
* Enterprise Edition
* Business Critical Edition (prev. ESD = Enterprise for Sensitive Data)
* Virtual Private Snowflake (VPS)



### Standard Edition does not have

* Query acceleration
* Search optimization
* Materialized views
* `ACCESS_HISTORY`
* Synthetic data generation
* 90+ days **Time Travel**
* Periodic rekeying of encrypted data
* Row & Column level security
* Multi Cluster Virtual Warehouse


### Features unique to BCE & VPS

* Amazon API Gateway 
* Failover & fallback between Snowflake accounts
* **Tri-Secret Secure** customer managed encryption keys
* PCI DSS
* PHI
* IRAP
* Private Link

### Features unique to VPS

* Dedicated metadata store & compute cluster pool

## Snowflake Releases

### Weekly

1. Full Release
2. Patch Release


### Monthly

* Behavior change
* Typically as part of 3rd or 4th weekly release of the month


### Bundle Lifecycle

1. 1st month: Testing period = Disabled by default
2. 2nd month: Opt-out period = Enabled by default
3. 3rd month: Generally Enabled (cannot be disabled)

