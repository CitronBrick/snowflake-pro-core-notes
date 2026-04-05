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
* Temporary  / Transient tables are charged when they're alive


##### Storage costs : Database

* based on file size
* Though Continuous Data Protection (Time Travel & Fail safe) by itself is free, it impacts Storage cost
* Active, Time Travel, Fail-Safe states are charged
* `INSERT`, `COPY`, `SNOWPIPE` commands impact `TIME_TRAVEL_BYTES` & `FAILSAFE_BYTES` charges due to micropartition destruction & recreation
* View costs
	* Snowsight: Catalog -> Database Explorer -> <Database> -> Tables
	* `SHOW TABLES`
	* best:  `TABLE_STORAGE_METRICS` view in Snowflake Information Schema & `ACCOUNT_USAGE` 

##### Storage costs: Hybrid tables

* charged Gb/month
* row store copy is **costlier** than traditional Snowflake storage
* CDP same charges as Standard tables




##### Storage costs: Time Travel & Fail safe

* calculated for each 24 hours since data change
* nb of days depends on table type & Time travel retention period
* only delta information is stored (like git) => only a % of the table changed is charged
* table drop & truncation => full table size charge


##### Storage costs: Cloning costs

* 0 copy clone does not consume storage costs initially for standard tables
* subsequent changes & CDP consume storage
* Hybrid table cloning consumes both storage & compute costs

##### Storage costs: Cross cloud Auto fulfillment

* provides data to other cloud regios without manual data replication
* data transfer cost
* Egress Cost Optimizer saves cost


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

* Database & Share replication : all accounts
* Other object replication & Failover : BCE+




### Replication group 

* defined object collection in source account that are replicated to 1+ target accounts within same organization
* readonly in target

### Failover group

* replication group that can also failover
* secondary failover groups are read only in target
* primary failover group is read-write in target
* any secondary can be promoted to primary in target

### Data transfer costs

* data transfer due to initial & subsequent sync operations are charged by cloud provider
* applied even if transfer

### Compute costs

* Snowflake compute is used for 
	1. metadata & data delta calculation 
	2. for actual copy


### Storage costs

* stardard storage costs in target account

### Misc 

* costs are applied even if transfer fails. 
* Partial data copied stays in target for 14 days
* In target account, bg auto tasks for materialized views & search optimization also consume credits

### Check costs

* Snowsight: Admin -> Cost management 
* `information_schema.replication_group_usage_history`
```
SELECT (end_time - start_time) as duration, credits_used, bytes_transferred
FROM TABLE(
	information_schema.replication_group_usage_history(
		date_range_start => DATE_ADD('date', -7, CURRENT_DATE())
	)
);
```
* `SNOWFLAKE.ACCOUNT_USAGE.REPLICATION_GROUP_USAGE_HISTORY`

```
SELECT (end_time - start_time) as duration, credits_used, bytes_transferred
FROM snowflake.account_usage.replication_group_usage_history
WHERE start_time >= DATE_TRUNC('month', CURRENT_DATE());
```


## Attributing cost

* attribute cost to different departments within an organization
* **Object tags** : associate resources and users with departments
* **Query tags**: associate queries with departments (when same app queries on behalf of users belonging to multiple departments)
* Scenarios
	* Resources used by 1 department: use object tags on warehouses to attribute costs to department
	* Resource are shared by users from multiple departments: use object tags on users to associate user <-> department.
	* Apps/Wokflows shared by users from multiple departments: query tag for each query which identifies the user's department.
* Workflow
	1. Create tags
	2. Replicating tag database
	3. Tagging resources & users

### Create tags

```
use role tag_admin;
create database cost_managmeent;
create schema tags;

create tag cost_center ALLOWED_VALUES 'finance', 'marketing', 'engineering', 'product';
```

#### Replicating tag database (transfer tags to different account in same organization)

* In source account 

``` 
CREATE REPLICATION GROUP cost_management_repl_group
	OBJECT_TYPE = DATABASES
	ALLOWED_DATABASES = cost_management
	ALLOWED_ACCOUNTS = my_org.acct_1, my_org.acct_2
	REPLICATION_SCHEDULE = '10 MINUTE';
```

* In each target  account

Create secondary replica group & refresh from primary group

```
CREATE REPLICATION GROUP cost_management_repl_group 
	AS REPLICA OF my_org.my_acct.cost.cost_management_repl_group;

ALTER REPLICATION GROUP cost_management_repl_group REFRESH;
```

#### Tagging resources/ users

``` 
ALTER WAREHOUSE wh1 SET TAG cost_management.tags.cost_center = 'SALES';
ALTER WAREHOUSE wh2 SET TAG cost_management.tags.cost_center = 'FINANCE';

ALTER USER finance_user SET TAG cost_management.tags.cost_center = 'FINANCE'
```

### Viewing cost by tag


#### Viewing cost by tag within account

##### TAG_REFERENCES 

* 2 hour latency for view
* tag inheritance not included

```
SELECT object_database||'.'||object_schema||'.'||object_name as obj
FROM SNOWFLAKE.ACCOUNT_USAGE.TAG_REFERENCES
WHERE TAG_NAME = 'cost_center' AND TAG_VALUE='FINANCE';
```

##### WAREHOUSE_METERING

* upto 365 DAYS
* latency 3 hours

```
SELECT sum(credits_attributed_compute_queries) AS non_idle_wh_time
FROM SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_METERING
GROUP BY warehouse_name;
```

##### QUERY_ATTRIBUTION_HISTORY

* upto 365 days
* latency upto 8 hours

```
SELECT query_id, query_tag, credits_attributed_compute 
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_ATTRIBUTION_HISTORY;
```