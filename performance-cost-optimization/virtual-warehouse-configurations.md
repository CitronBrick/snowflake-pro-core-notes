# Virtual Warehouse configurations

## Warehouse types

* Standard
* Snowpark optimized
* Multicluster 
* Interactive

### Standard

Gen 1 & Gen 2 warehouses.
Gen 2 not available in :
* AWS EU (Zurich)
* AWS Africa (Cape Town)
* GCP Africa (Dammam)
* Azure US Gov Virginia (FedRAMP High Plus)
* Azure US Gov Virginia 

`CREATE OR REPLACE WAREHOUSE wh2 RESOURCE_CONSTRAINT= STANDARD_GEN_2`


### Interactive 

* optimized for low-latency interactive workloads
* uses additional metadata & indexes of underlying interactive tables to accelerate queries
* run continuously
* serves high volume of concurrent queries

```
CREATE OR REPLACE INTERACTIVE WAREHOUSE interactive_demo
TABLES (my_orders)
WAREHOUSE_SIZE='XSMALL'
```

#### Limitations of Interactive warehouses

* long running queries not supported
* `SELECT` < 5s
* not auto-suspended when idle. manual suspension => significant latency
* Standard tables cannot be queried

### Snowpark optimized

* Better for Snowpark workloads requiring
	* large memory 
	* specific CPU configuration
	* Eg:
		* ML using stored procedure on single virtual warehouse load
		* Snowpark workloads using UDF or UDTF
* no benefit for non-Snowpark workloads
* Default memory : Standard * 16

```
CREATE WAREHOUSE so_warehouse WITH
	WAREHOUSE_SIZE = 'LARGE'
	WAREHOUSE_TYPE = 'SNOWPARK-OPTIMIZED'
	RESOURCE_CONSTRAINT = 'MEMORY_16X_X86'
```

## Multi-cluster warehouse

* max nb cluster > 1
* In Snowsight MAX_CLUSTER_COUNT is 10
* For `MAX_CLUSTER_COUNT` > 10, use `CREATE WAREHOUSE` or `ALTER WAREHOUSE`
* 2 modes
	* Maximized
	* Auto-Scale
		* 2 policies (policies are applicable only for Auto-Scale mode)
			* Standard (default)
			* Economy

### Maximized

* min cluster = max cluster
* when large number of concurrent user sessions/queries
* numbers don't fluctuate

### Auto-Scale

* min cluster < max cluster
* clusters are started/stopped as needed

	

| Policy 	| Description 				| New cluster starts when 					| cluster shutdown when 				|
|---------	| ----------------			| ---------------------------------------	| ------------------------------------	|
| Standard	| Prefers cluster creation	| MAX_CLUSTER_COUNT <= 10 ? +1 : +many		| MAX_CLUSTER_COUNT <= 10 ? -1 : -many	|
| Economy	| Prefers credits saving	| only if enough load for 6+ minutes		| if load <  6min MAX_CLUSTER_COUNT <= 10 ? -1 : -many  |

* The least loaded cluster is shutdown after all queries are done

## Warehouse sizing

* **Data loading performance** depends on number & size of files, *not* necessarily on warehouse size
* Larger warehouses improve query time

| Warehouse size | Gen 1: Credits/hour		| Notes 										|
| X-Small		 | 1						| default for Snowsight & `CREATE WAREHOUSE`	|
| Small			 | 2						|												|
| Medium		 | 4						|												|
| Large		 	 | 8						|												|
| X-Large		 | 16						| default for Snowsight							|
| 2X-Large		 | 32						|												|
| 3X-Large		 | 64						|												|
| 4X-Large		 | 128						|												|
| 5X-Large		 | 256						| AWS & Azure. Preview for US Gov regions		|
| 6X-Large		 | 512						| AWS & Azure. Preview for US Gov regions		|


## Warehouse settings

* `AUTO_SUSPEND`
	* nb of seconds before automatic suspension of warehouse
	* 0 or `NULL` not recommended => too costly
	* background suspension process runs ~ every 30s => **not** precise
	* values that are not multiples of 30 => allowed but may have unexpected behavior
	* Default : 10min => 600
* `AUTO_RESUME`
	* TRUE : (default) warehouse resumes for new query
	* FALSE : needs `ALTER WAREHOUSE` or explicit UI action
* `INITIALLY_SUSPENDED`
	* TRUE : suspended when created
	* FALSE : (default) running when creted
* `RESOURCE_MONITOR`
* `COMMENT`
* `TAGS`
* `ENABLE_QUERY_ACCELERATION`: default FALSE
* `QUERY_ACCELERATION_MAX_SCALE_FACTOR`: 
	* default 0 : no limit
	* valid values : 0-100
	* used as multiplying factor based on warehouse size
* MAX_CONCURRENCY_LEVEL : (this & following params are optional in `create warehouse`)
	* after this point
		* single cluster or maximized multi-cluster: query is queued
		* auto-scale multi-cluster : new cluster(s) started
* `STATEMENT_QUEUED_TIMEOUT_IN_SECONDS`
* `STATEMENT_TIMEOUT_IN_SECONDS`


## Warehouse access control

| Privilege					| Object 		| Notes 																|
| ----------------------	| --------		| ---------------------------------------------------------------------	|
| CREATE WAREHOUSE 			| Account 		| only SYSADMIN or higher has this privilege by default. can be granted to additional roles |
| OWNERSHIP					| Warehouse 	| Required to run `create or alter warehouse` on existing wh. can be granted to others via `grant ownership`	|




