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