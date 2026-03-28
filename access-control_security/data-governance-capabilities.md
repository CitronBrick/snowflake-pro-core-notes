# Data Governance capabiliteis 


## Accounts

* **tags** can be set on account level
* Cortex code includes in-built data governance skilles



### Trust Center

* evaluates account against recommendations defined by **Scanners**
* Snowflake reader accounts are not supported
* use `GLOBALORGADMIN` instead of `ACCOUNTADMIN` to grant the Trust Center roles


### Session Usage

* using Snowsight = 
* `SNOWFLAKE.ACCOUNT_USAGE.SESSIONS` view 
* idle session timeout
	* `SESSION_UI_IDLE_TIMEOUT_MINS` for Snowsight
	* `SESSION_IDLE_TIMEOUT_MINS` for others
	* min 5 min, default 4 hours
	* if `CLIENT_SESSION_KEEP_ALIVE` is true, session does not timeout (bad)


## Organizations

