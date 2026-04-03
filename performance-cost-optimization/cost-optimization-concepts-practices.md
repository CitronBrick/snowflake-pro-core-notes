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
