# Cost Optimization concepts & best practices

## Understand & explore costs

### Snowight cost insights

### Cost Insights steps

1. Sign in to Snowsight
2. Switch to role with cost related feature access
3. Navigation menu -> Admin -> Cost management
4. Account overview tab
5. Cost insights tile


#### Insights

1. rarely used tables with automatic clustering => `Alter Table t1 Suspend Recluster`
2. rarely used materialized views => `DROP MATERIALIZED VIEW mv1`
3. rarely used search optimization paths => `ALTER TABLE t1 DROP SEARCH OPTIMIZATION`
4. large tables that are never queried => `DROP TABLE t1`
5. Tables > 100Gb with mostly writes => `DROP TABLE t1`
6. short lived permanent tables => `CREATE TRANSIENT TABLE t1`
7. Inefficient usage of multi-cluster warehouses => `ALTER WAREHOUSE wh1 SET MIN_CLUSTER_COUNT = 1`

For the remedies 1-5, consider:
1. If table is used for Disaster Recovery
2. If table is used for Data sharing to other Snowflake accounts


#### Search Optimization paths (EE+)

* improves lookup of lookup & analytical queries
* search access path tracks which column values reside in which micropartitions
* maintenance service maintains after DML shown, as in `search_optimization_progress` of `SHOW TABLES`
* `SELECT SYSTEM$ESTIMATE_SEARCH_OPTIMIZATION_COST('<table_without_search_opt>')`

##### Cost reduction

* choose tables & columns for search optimization
* `delete` old data for tables that are supposed to contain recent data only
* batch together `insert`, `merge` , `update`
* reclustering
	1. drop the search optimization property
	2. recluster
	3. add back search optimization property
* substring & variant equality: evaluate `SYSTEM$ESTIMATE_SEARCH_OPTIMIZATION_COST` beforehand

#### Storage costs

* Include
	* Staging files (compressed or uncompressed) (price based on file size) (no CDP based charges)
	* Database files 
	* Fail safe for tables
	* Clones
* flat rate / Tb
* Amount charged depends on
	* region (US or EU)
	* account type (Capacity or On-demand)

##### Storage costs : Database

* based on file size
* Though Continuous Data Protection (Time Travel & Fail safe) by itself is free, it impacts Storage cost
* Active, Time Travel, Fail-Safe states are charged
* `INSERT`, `COPY`, `SNOWPIPE` commands impact `TIME_TRAVEL_BYTES` & `FAILSAFE_BYTES` charges due to micropartition destruction & recreation
* View costs
	* Snowsight: Catalog -> Database Explorer -> <Database> -> Tables
	* `SHOW TABLES`
	* best:  `TABLE_STORAGE_METRICS` view in Snowflake Information Schema & `ACCOUNT_USAGE` 

##### Storage costs: Time Travel & Fail safe

* calculated for each 24 hours since data change
* nb of days depends on table type & Time travel retention period
* only delta information is stored (like git) => only a % of the table changed is charged
* table drop & truncation => full table size charge


##### Storage costs: Cloning costs

* 0 copy clone does not consume storage costs initially for standard tables
* subsequent changes & CDP consume storage
* Hybrid table cloning consumes both storage & compute costs

#### Compute Costs

* Include
	* Virtual warehouse compute
	* Serverless compute
	* Compute pools
	* Cloud services compute
* `SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY` view tracks cost of above except Serverless compute
* Serverless Compute
	* `SNOWFLAKE.ACCOUNT_USAGE.SNOWPARK_CONTAINER_SERVICES_HISTORY` 
	* `select * from SNOWFLAKE.ACCOUNT_USAGE.DAILY_METERING_HISTORY where service_type='SNOWPARK_CONTAINER_SERVICES'`
	* `select * from SNOWFLAKE.ORGANIZATION_USAGE.DAILY_METERING_HISTORY where service_type='SNOWPARK_CONTAINER_SERVICES'`
	* `select * from SNOWFLAKE.ACCOUNT_USAGE.METERING_HISTORY where service_type='SNOWPARK_CONTAINER_SERVICES'`


##### Virtual warehouse compute costs

* Snowflake credits are consumed when **running** (even **without** any queries) not when **suspended**
* Even idle or underutilized virtual warehouses consume credits !!
* hourly rate, billed per second (1 min minimum, then every second)
* when warehouse is **resized**, only **additional** resources are billed

##### Serverless compute cost

* Snowflake managed compute resources that are automatically resized / scaled up.
* Charges based on usage only
* Computer hours billed per second rounded to nearest second

##### Compute pool compute cost

* collection of 1+ VM nodes on which Snowpark Container Services are run.
* number & type of instances determine cost
* charges incurred: `IDLE`, `ACTIVE`, `STOPPING`, `RESIZING` states
* free : `STARTED`, `SUSPENDED` states

##### Cloud services compute cost

* billed only if : cloud service consumption > 10% of daily warehouse usage


## Trans-region data transfer Cost considerations


## Replication

