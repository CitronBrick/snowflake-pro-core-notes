# Continuous Data protection


## Time Travel

* allows:
	* querying past (deleted/modified) data
	* clone tables/schemas/databases past versions
	* restore tables/schemas/databases
* SQL extensions:
	* SELECT .... AT
	* SELECT .... BEFORE
	* CREATE CLONE ..... AT
	* CREATE CLONE ..... BEFORE
	* UNDROP {object}
* Retention period
	* default : 1 day
	* Standard Edition account: 0 - 1 day
	* Enterprise Edition account: 0 - 90 days
	* Retention period: 0 => Time travel disabled
	* effective account level min data Retention period = MAX(DATA_RETENTION_PERIOD_IN_DAYS, MIN_DATA_RETENTION_PERIOD_IN_DAYS)
* Limitations
	* Following are not cloned using Time Travel
		* External tables
		* Internal Snowflake staging tables
		* Hybrid tables for schemas (possible for databases)
		* Users tasks in database  are not cloned during schema cloning

## Fail-safe

* disaster recovery of historical data (by Snowflake)
* for operational-failures
* non-configurable 7 day period
* *last resort*  
* *best effort*
* 7 day *after* Time Travel retention period
* Snowpipe Streaming Classic doesn't support fail-safe => No fail-safe for ingested data 
* View
	1. need ACCOUNTADMIN role
	2. Admin
	3. Cost management
	4. Consumption
	5. Usage Type : Storage
	6. Storage breakdown


## Data encryption

a. Customer Provided Staging area

Customer (Corporate Network) -> Staging Area (Supported Cloud Storage) -> Snowflake (Snowflake VPC) -> Staging Area (Supported Cloud Storage) -> Customer (Corporate Network)

b. Snowflake Provided Staging area

Customer (Corporate Network) -> Snowflake Staging Area -> Snowflake (Snowflake VPC) -> Snowflake Staging Area -> Customer


1. Loading data files to storage area
	a. For External  staging area, Client-side encryption by user (optional)
	b. For Internal staging area, Snowflake encrypts data automatically on client machine's Snowflake client
2. External Staging area to Snowflake VPC, automatic encryption by Snowflake
3. Unloading results to 
	a. For External staging area, Client-side encryption by user (optional)
	b. For Internal staging area, automatic encryption by Snowflake
4. The user downloads the data files from stage
5. The user decrypts files on the client side


## Cloning

* If source is database or schema, the clone inherits all granted privileges on the clones of all child objects
* clone of container (database or schema) does not inherit privileges of source container
* cloning tables & views (support COPY GRANTS parameter), copies all privileges except OWNERSHIP from source to target
* Other objects (that don't support COPY GRANTS parameter)
* `CREATE SCHEMA CLONE ... WITH MANGED ACCESS`
	* if source is Managed => `OWNERSHIP` is required on the source
	* if source is not managed => `MANAGE GRANTS ON ACCOUNT` & `USAGE` are required on the source schema

* Object parameters
	* cloned objects inherit object parameters set on source object. (already set on source when cloning)
	* object parameters set on object containers & not explicity set on cloned object => object clone inherits default parameter value or value overriden at the lowest level

* Sequences
	* if database or schema containing both table & sequence is cloned => cloned table references cloned sequence
	* else => cloned table  references the source sequence
		* override using `ALTER TABLE table_name ALTER COLUMN column_name SET DEFAULT new_sequence NEXTVAL`

* Foreign key constraint
	* table with fk constraint is cloned => cloned table refers source or cloned table with primary key
	* database/schema containing both tables is cloned => cloned table with fk constraint refers pk in cloned table
	* tables are in different database/schema => cloned table with fk constraint refers to primary key in source table

* Cloning & clustering keys
	* Post-cloning, by default *Automatic clustering* is **suspended** for the new table
	* `ALTER TABLE table_name RESUME RECLUSTER` to resume clustering

* Cloning & stages
	* cloning external staging does not impact referenced external storage
	* cloning database/schema => external named stages (already present) are cloned
	* cloning tables => table's internal stage is cloned (data files are **not** cloned => cloned table stages are **empty** )
	* cloning named internal stages
		* use `INCLUDE INTERNAL STAGES` clause
		* supported only at database/schema level
		* for stages with **directory table** enabled:
			* **empty** clones are created (files not copied)
		* clones use *current* state of internal named stage, *irrespective* of Time Travel. 
			* If a point in time before stage creation is used => no cloning
		* relies on `COPY FILES` service => use `COPY_FILES_HISTORY` view to monitor credit usage 

* Cloning Apache Iceberg tables
	* DML operations on cloned tables, new data files are stored in **source** table's base location, w/o affecting the source tables
	* Cloned tables have their own metadata files. Eg: metadata.json file with unique `table-uuid`, `last-sequence-number`
	* Cloned tables backup do not include any source table backup info.
* Cloning Event tables
	* Clone from Event table <=> Clone to Event table
	* Clone from regular to Event table => impossible
	* Clone from Event table to regular table => impossible
* Cloning Tasks & Alerts
	* Database/schema is cloned => Tasks & Alerts in new clone are *suspended* by default
	* `ALTER [TASK/ALERT] ... RESUME` to resume
* Cloning Governance objects
	* Masking & row access policies
		* cloning an individual policy is not supported
		* cloning schema => clone all policies in schema
		* cloning table => clone all table policies
			* table policy refers to table/view in same database/schema => cloned table policy refers to table/view in the target database/schema
			* table policy refers to table/view in different database/schema => cloned table policy refers to the Foreign table
		* External tables
			* `VALUE` column
				* cannot have a masking policy during `CREATE EXTERNAL TABLE`
				* use `ALTER TABLE t1 MODIFY COLUMN VALUE SET MASKING POLICY p1`	 
			* Virtual columns
				* set `EXEMPT_OTHER_POLICIES=TRUE` on the masking policy of the `VALUE` column => overrides default inherited by Virtual columns.
				* set different policy for Virtual column using `ALTER TABLE`

