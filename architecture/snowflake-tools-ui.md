## Snowflake tools

### Snowsight

* Snowflake web interface
* run SQL or Python
* E2E machine learning using **Snowflake Notebooks**
* **Snowflake Copilot** for data processing
* query performance & history
* Worksheets
* **Streamlit** apps
* Snowflake Native apps
* **Trust Center**

### SnowSQL

* legacy -> Snowflake CLI
* open source
* CLI
* execute commands for
	* Streamlit
	* **Snowpark Container Services**
	* Snow Native App Framework

### Snowflake connectors

* native integration of 
	* 3rd party apps
	* database systems
* no need to manually integrate against API endpoints
* instant access to current data
* supports both
	* initial laod of historical data
	* incremental changes
* Snowflake Connector for ...
	* Google Analytics Aggregate Data
	* Google Analytics Raw Data
	* Google Looker Studio
	* ServiceNow
	* MySQL
	* PostgreSQL
	* Sharepoint

### Snowflake drivers

* Golang
* JDBC
* .NET
* ODBC
* PHP PDO Driver
* Python
* TLS support: 1.2 & 1.3


### SnowPark 

* set of libraries & code environment that run Apache Spark in the Snowflake vectorized engine



### SnowCD

* Snowflake Connectivity Diagnostic Tool
* uses results from `SYSTEM$ALLOWLIST()` or `SYSTEM$ALLOWLIST_PRIVATELINKE()`
* Detects the following:
	* No HTTP server running at specified IP & port
	* DNS lookup failure
	* MITM attack
	* certain other failures below HTTP level
* Limitations:
	* stages require additional auth info that SnowCD doesn't have => no strict check on HTTP response code from stage
		* Access denial policy for S3, Azure Blob Storage or Google Cloud Storage
		* problem connecting with customer's proxy server

### Streamlit

* Open source Python library
* facilitates custom webapp creation & sharing for machine learning & data sicne
* Snowflake manages underlying compute & storage for Streamlit apps
* Streamlit apps are Snowflake objects => RBAC
* Streamlit apps use internal stages to store files & data
* Streamlit apps work seamlessly with 
	* Snowpark
	* UDFs
	* stored procedures
	* Snowflake Native apps
* Using with Snowsight, side-by-side editor & app preview screen, to add/remove components
* A single Virtual Warehouse is required to run Streamlit app & SQL queries
* Virtual Warehouse is active as long as the app's WebSocket is active.
* WebSocket connection expires  15min after last usage
* Conserve credits by suspending Virtual Warehouse

### Cortex

* suite of features using LLM to
	* understand unstructured data
	* answer freeform questions
	* provide intelligent assistance
* includes
	* Cortex Agents
	* Snowflake Cortext AI Functions (including LLM functions)
	* Cortex Analyst
	* Cortex fine tuning
	* Cortex Search
	* Document AI
	* Snowflake Copilot
	* Snowflake intelligence

### Snowflake SQL API

* REST API
* perform queries
* manage deployment 
	* provision roles/users
	* create table
* check query execution status
* cancel execution
* authencation
	* OAuth
	* kep pair


