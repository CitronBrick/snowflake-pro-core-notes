# Concepts & best practices when loading data

## Stages

A stage specifies where data files are stored (staged) so that the data can be loaded to a table

1. Internal Stage
	* User
	* Table
	* Named
2. External Stage
	* AWS S3 buckets
	* Google Cloud Storage buckets
	* Microsoft Azure Containers


### Internal Stages

* must be specified for `PUT`
* same stage must be specified for corresponding `COPY INTO <table>` command
* each user & table is automatically assigned an internal stage
* in addition, **Named Internal Stage** can be created

#### Named Stage

* can be altered / dropped
* users with appropriate privileges can load data into any table
* recommended for regular loads with multiple users & multiple tables
* is a database object (roles, ownership transfer etc)
* `CREATE STAGE my_int_stage ENCRYPTION = (TYPE = 'SNOWFLAKE_SSE' )`
 
#### User Stage

* assigned by default to user
* convenient when : data accessed by single user will be loaded to multiple tables 
* referenced using `@~`
* `LIST @~` lists all files in user stage
* cannot be altered drop
* do not support file format options (can be set in `COPY INTO <table>` instead)
* unsuited for
	* multiple users need to access files
	* current user does not have `INSERT` privileges on the tables

#### Table Stage

* allocated by default to each table
* not supported for Apache Iceberg tables
* has the same name as table : `@%<exact_table_name>`
* not a separate database object (implicit) => no grantable privileges of its own.
* not appropriate to load into multiple tables
* cannot be altered/dropped
* do not support transformation while data is being loaded
* to query/list/drop files, `OWNERSHIP` on table is needed




