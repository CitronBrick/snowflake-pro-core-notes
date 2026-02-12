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