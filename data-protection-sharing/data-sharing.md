# Data sharing

## Account types

* Organization account
	* special account used by org admin to manage multi-account organizations.
	* They can access usage data from premium views in `ORGANIZATION_USAGE` schema.
* Regular account (includes trial accounts)
* Snowflake Open Catalog account
	* Special account used by service admins & catalog admins to manage **Snowflake Open Catalog**

## Snowflake Marketplace

* data
* toolkits
* apps
* AI agents


### Providers

* publish listings for free to use datasets
* publish listings with samples of datasets, upon request from specific consumer
* share live datasets securely & in *real-time* w/o creating copies or imposing data integration tasks on consuemr
* share public listings in Virtual Private Snowflake deployments (preview)
* Deliver data to customers w/o building APIs or pipelines

### Consumers

* discover & test 3rd party datasets
* receive raw data from vendors
* new datasets + existing Snowflake data = business insights
* Eliminate cost of building APIs / pipelines
* use BI tools of your choice

## Snowflake Data Exchange

* share data with **selected** group of **invited** consumers  (eg: internal departments, partners)
* hosting account is the Data Exchange Admin. With `ACCOUNTADMIN` can add both publishers & consumers
* Snowflake support needs to be contacted for Data Exchange	setup


## Access control options


### Create Share

```
CREATE OR REPLACE SHARE IF NOT EXISTS share_name COMMENT='my comment'
```

`CREATE SHARE` requires **ACCOUNTADMIN** by default. Privilege can be granted to additional roles.

### 


### Drop Share

```
DROP SHARE share_name
```

* `DROP SHARE` requires *OWNERSHIP*  privilages.
* Dropped shares can't be restored => only recreated.
* Even after re-creation, consumers must **re-create** databases from share

### Alter Share

```
ALTER SHARE [IF EXISTS] <name> { ADD | REMOVE } ACCOUNTS= <consumer-account>+ SHARE RESTRICTIONS= { TRUE | FALSE }

ALTER SHARE [IF EXISTS] <name> SET ACCOUNTS = <consumer-account>+ 

ALTER SHARE [IF EXISTS] <name> SET COMMENT = <consumer-account>+ 

ALTER SHARE [IF EXISTS] <name> SET TAG = <name=value>+ 

ALTER SHARE [IF EXISTS] <name> UNSET COMMENT

```

* Privileges needed:	
	* *OWNERSHIP*
	* *MANAGE SHARE TARGET*
* `SHARE_RESTRICTIONS = FALSE` : 
	* Standard or Enterprise account can be added to Business Critical Provider account's share  
	* non-HIPAA account can be added to HIPAA compliant provider account's share
* `SHARE_RESTRICTIONS = TRUE` :
	* Standard or Enterprise account cannot be added to a Business Critical Provider account's share
	* non-HIPAA account cannot

## Secure Data Sharing

| Data sharing mechanism			| Listing 					| Direct share 				|
|---------------------------------	| -------------------------	| ------------------------	|
| Share with whom ?					| 1+ account in any region	| 1+ account in your region	|
| Auto fulfill across clouds		| Yes						| No 						|
| Optionally charge for data 		| Yes 						| No 						|
| Optionally offer data publicly 	| Yes 						| No 						|
| Get Consumer usage metrics		| Yes 						| No 						|

### Direct share

* share data with 1+ account in same Snowflake region
* No copy/move data
* can be converted to listing

### Data listing

* provider -> consumer model
* share Snowflake Native app / data
* can be shared with specific accounts or Snowflake Marketplace
* can include metadata: title, description, sample SQL queries, & data provider info
* can be charged
* pricing: free, limited free tier (1 - 90 days), paid
	

