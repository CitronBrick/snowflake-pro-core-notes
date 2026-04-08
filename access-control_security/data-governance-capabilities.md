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

## Secure Views

* Non-secure views can expose underlying table structure, which may not be desirable
* Non-secure views' internal optimizations might expose underlying data through UDFs 
* Secure views hide underlying table structure & do not have internal optimizations
* Secure view internals are hidden in the **Query Profile even for owners** (non-owners may have access to owner's Query Profile)
* Secure views should not be used for query convenience views
* Secure views can have redacted error messages
* To create: `CREATE SECURE VIEW` or `CREATE SECURE MATERIALIZED VIEW` 
* To make unsecure : `ALTER VIEW <viewname> UNSET SECURE` or `ALTER MATERIALIZED VIEW <viewname> UNSET SECURE`
* To make secure : `ALTER VIEW <viewname> SET SECURE` or `ALTER MATERIALIZED VIEW <viewname> SET SECURE`


### Make non secure views expose data

1. Consider an user who has access to only view red widgets

2. `SELECT * from widgets WHERE 1/IFF(color='Purple',0,1) = 1;`

3. The above code will fail with a 0 division error if there are purple widgets, making the user aware of their presence.




### Check if view is secure

* Information Schema
	```
	SELECT table_catalog, table_schema, table_name, is_secure
	FROM my_db.information_schema.views WHERE table_name='MyView';
	```
* Account_Usage
	```
	SELECT table_catalog, table_schema, table_name, is_secure
	FROM snowflake.account_usage.views WHERE table_name='MyView';
	```
* Show
	```
	SHOW VIEWS LIKE 'my_view';
	SHOW MATERIALIZED VIEWS LIKE 'my_view';
	```


#### View Secure views Definition

* Not possible via:
	* `SHOW VIEWS` / `SHOW MATERIALIZED VIEWS`
	* `GET_DDL('VIEW','my_db.my_schema.my_view')`
	* `information_schema.views`
* Posssible:
	* In `ACCOUNT_USAGE.VIEWS`, users with any of the following privileges can view
		* `IMPORTED PRIVILEGES` on any shared database (eg: `SNOWFLAKE`)
		* `ACCOUNTADMIN` role
		* `SNOWFLAKE.OBJECT_VIEWER` role (least privilege) 


## Secure UDFS & Stored Procedures

* Visible for unauthorized
	* Parameter types
	* Return type
	* Handler language
	* Null handling
	* Volatility
* Hidden for unauthorized
	* body
	* import list
	* handler name
	* packages list
* Hides from
	* `SHOW FUNCTIONS`
	* `SHOW USER FUNCTIONS`
	* `DESCRIBE FUNCTION`
	* Information schema functions
	* `SHOW PROCEDURES`
	* `DESCRIBE PROCEDURE`
	* Information schema procedures
	* Query Profile
	* `GET_DDL`

### Pushdown & leaking

## Access history

* requires EE+
* tracks
	* `INSERT`
	* `UPDATE`
	* `DELETE`
	* `COPY`
* `ACCESS_HISTORY` view in `ACCOUNT_USAGE` & `ORGANIZATION` schemas
* 1 row / SQL statement
	* source columns
	* projected columns
	* unprojected columns (eg: `where` columns absent in `select`)
	* user name
	* `ORGANIZATION` schema's `ACCESS_HISTORY` has additional info on 
		* Organizational listing
