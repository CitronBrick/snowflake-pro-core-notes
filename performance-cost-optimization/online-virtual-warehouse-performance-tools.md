# Online Virtual Warehouse Performance Tools

## Resource Monitor

* help to control credit usage by running warehouses & their cloud services
* nb credits depends on size & running duration of warehouse
* Credit usage limits can be set for specified interval or date range
* Resource Monitor can trigger actions the following once limit reached.
* ACCOUNT_ADMIN roles is needed to create Resource monitor
* FREQUENCY & START_TIMESTAMP can be specified. (default: `MONTHLY` & `IMMEDIATELY`)

### Actions

* **Notify** : send notification only
* **Notify & Suspend** : suspends warehouse after all running statements are completed
* **Notify & Suspend immediately** : cancels running statements & suspends warehouse

Non-admins can receive notifications only for Warehouse Resource monitors


## Scaling up vs Scaling out

* Scale up by resizing warehouse
* Scale out by adding clusters to multi-cluster warehouse (requires Enterprise Edition +)
* resizing helps with running larger more complex queries
* adding warehouses helps with concurrency issues
* resizing running warehouse is possible but is applied only for next queries
* resizing 5XL or 6XL warehouse to 4XL results in brief period where both warehouses are charged
* for Snowflake Enterpise Edition+, all warehouses should be multi-cluster preferably in Auto-Scale 
* for multi-cluster keep min value 1 (clusters are started as needed only), unless high availability is a concern 
* for multi-cluster  max value as big as possible


## Query Acceleration Service 

* Enterprise Edition+
* Supported queries
	* `SELECT`
	* `INSERT`
	* `CREATE TABLE AS SELECT`
	* `COPY INTO <table>`
* might increase credit consumption rate of warehouse
* `QUERY_ACCELERATION_MAX_SCALE_FACTOR`
* `QUERY_ACCELERATION_ELIGIBLE` view
* `SYSTEM$ESTIMATE_QUERY_ACCELERATION` function


```
CREATE WAREHOUSE my_wh WITH ENABLE_QUERY_ACCELERATION = true;

ALTER WAREHOUSE my_other_wh WITH ENABLE_QUERY_ACCELERATION = true;


SHOW WAREHOUSES LIKE 'my_wh'
	-->> SELECT "name", "enable_query_accleration", "query_acceleration_max_scale_factor" FROM $1
```


### Eligible queries for QAS

* 2 patterns
	* large scans with an aggregation or selective filter
	* large scans adding many new rows (`INSERT` or `COPY INTO`)
* Snowflake does not have specific cutoff
* Eligibility depends on query plan & warehouse size
* Snowflake increases eligibility net over time
	* previously `LIMIT` & `ORDER BY` were ineligible
	* now eligible


### Factors for QAS ineligibility

* insufficient partitions in the scan
* un-selective filters
* high cardinality of `GROUP BY`
* non-deterministic results. Eg: `SEQ`, `RANDOM`

```
SELECT query_id, eligible_query_acceleration_time
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_ACCELERATION_ELIGIBLE
WHERE start_time > DATEADD('day', -7, CURRENT_TIMESTAMP())
ORDER BY eligible_query_acceleration_time DESC;
```

```
SELECT parse_json(SYSTEM$ESTIMATE_QUERY_ACCELERATION('abc-def'))

```

### SCALE FACTOR

* default value: 0 => Infinity
* scale factor 5 for medium warehouse
	* warehouse can lease compute resources upto 5 times the medium warehouse size
	* medium warehouse: 4 credits/hr
	* => 20 credits/hr
* scale factor is applied to **entire warehouse**, irrespective of single/multi clustered
	* for multi cluster warehouse => increase scale factor
* cost
	* credits are billed separately from warehouse usage
	* billed by second when in service is in use
	* cost is same irrespective of nb of queries using the service simultaneously
* QAS uses only as many resources are needed & are available 


### QAS columns in QUERY_HISTORY view


```
	SELECT query_id, query_text,
		query_acceleration_bytes_scanned,
		query_acceleration_partitions_scanned
		query_acceleration_upper_limit_scale_factor
	FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY 
	WHERE start_time >= DATEADD(hour, -24, current_timestamp())
```

* `bytes_scanned`, `query_acceleration_bytes_scanned`, `partitions_scanned, `query_acceleration_partitions_scanned` increase during QAS usage.
* QAS creates intermediary results which increases the nb of bytes & partitions