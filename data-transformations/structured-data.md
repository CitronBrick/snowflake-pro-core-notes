# Data Transformations

## Work with Structured Data

### Estimation functions

* can be used as aggregate or window functions


#### Similarity

* MinHash algo is used to estimate similarity of 2 datasets
* MinHash calculates Jacard index **w/o** calculating neither union nor intersection => quick
* `MINHASH(k,expr)` returns MinHash state containing MinHash array of length `k`
* `MINHASH_COMBINE(state)` combines 2+ MinHash states into 1 output state
* `APPROXIMATE_SIMILARITY(state)` : returns 0 <= result <= 1 (1 identical, 0 no overlap)
* `APPROXIMATE_JACARD_INDEX(state)` : alias of `APPROXIMATE_SIMILARITY`

#### Frequent values

* Space-Saving algorthim [^1]
* `APPROX_TOP_K_COMBINE` uses Parallel Space Saving algorithm [^2]
* skew, counters directly propotional to accuracy
* `APPROX_TOP_K(col,[ k, counters] )` 
* `APPROX_TOP_K_ACCUMULATE` : skips the final step & returns summary state of the Space-Saving algorithm
* `APPROX_TOP_K_COMBINE`: combine multiple input states to 1 final output state
* `APPROX_TOP_K_ESTIMATE`:  cardinality estimate of state

#### Percentile values

* t-Digest algorithm [^3]
* constant relative error (no rigorous proof)
* `APPROX_PERCENTILE(numeric_col, percentile)` : returns number such n * p < result 
* `APPROX_PERCENTILE_ACCUMULATE` : skips final step. returns algo state
* `APPROX_PERCENTILE_COMBINE`: combine multiple input states to 1 final output state
* `APPROX_PERCENTILE_ESTIMATE`: percentile estimate of state


### Sampling

```
SELECT  ...
FROM  .... 
{ SAMPLE | TABLESAMPLE } [samplingMethod]
[ ... ]
```

```
samplingMethod ::= { { BERNOULI | ROW ( { <probability> | <num> ROWS} ) |
	{ SYSTEM | BLOCK } { <probability> } [ { REPEATABLE | SEED}  ] { <seed> } }}
```

* For BLOCK, nb of rows is not possible
* 0 <= probability <= 100
* BERNOULI (ROWS) is default
* 0 <= rows <= 1M
* SEED / REPEATABLE  is specific to BLOCK
* SEED cannot be used on views / subqueries
* nb of rows returned is an approximate value
* sampling the result of a join is possible only for BERNOULI
* Fraction based is faster than fixed size
* sampling w/o seed is faster than using seed
* BLOCK faster than BERNOULI
* table unchanged + seed unchanged = sample unchanged ( not true for copy )
[^1]: Efficient Computation of Frequent & Top-K elements in Data Streams: Metwally, Agrawal & Abbadi
[^2]: Cafaro, Pulimento & Tempesta
[^3]: Dunning & Ertl


### Supported function types

#### System functions

* control functions to execute system actions (eg. abort query)
* Information functions to return system info (eg: table's clustering depth)
* Information functions for query info (eg: EXPLAIN plans)

#### Table functions

* return multiple table rows for each individual input
* can return different number of rows each time 
* can be used in `FROM` clause
* can be used as input to another table function
* can be applied ot set of rows using `LATERAL` construct


#### External functions

* code stored outside Snowflake is called
* remotely executed
* security related info is stored in an API integration 
* Inside Snowflake, the External function is stored as a database object, containg the info needed to call the remote service
* must return a value
* can act like *scalar function* : 1 row returned per received row
* must accept & return JSON
* expose an HTTPS endpoint


##### External function call

1. User's client program sends SQL statement w/ external function call to Snowflake
2. Snowflake reads the external function definition & corresponding API integration info.
	a. External function definition includes
		* Proxy service URL
		* API integration name
	b. API integration info includes
		* Proxy service resource to use.
			* The resource contains info about remote service
		* Authentication info for the proxy service
3. Snowflake composes HTTP POST request with
	* JSON data to be processed
	* HTTP header info
	* Auth information (from API integration)
4. The HTTP request is sent to Proxy service
5. Proxy service receives POST, processes it & forwards to remote service
6. Remote service processes data & sends to proxy
	* if Remote service returns a HTTP code for *asynchronous processing* 
		* Snowflake sends 1 or more GET requests to retreive the result
7. Proxy returns result to User's client program

##### External functions: advantages


* can be written in GoLang or C# (not supported by UDFs) 
* can use 3rd party libraries not supported by UDFs
* can be consumed by both Snowflake & other software

#### External functions: limitations

##### External functions: limitations:  Creation-time

* administrator must do configuration work, requiring knowledge of the specific cloud platform
* Proxy service is needed
* may require regional or private endpoints 
* Only functions not stored procedures 
* Future grants of privileges not supported

##### External functions: limitations:  Execution-time

* not possible to optimize 
* more overhead => slower
* cannot be shared with customers via **Secure Data Sharing**
* security & data privacy issues 
* cannot be used in 
	* `DEFAULT` clause of `CREATE TABLE` statement
	* `COPY` transformation


| UDF 								| Stored Procedure 								|
|---------------------------------	| --------------------------------------------	|
| body required to return values	| body not required to return values			|
| calculate & return value   		| perform administrative task via SQL			|



### User Defined Functions (UDF) 

```
CREATE OR REPLACE FUNCTION addone(i INT)
	RETURNS INT
	LANGUAGE PYTHON
	RUNTIME_VERSION = '3.12'
	HANDLER = 'addone_py'
AS $$
	def addone_py(i):
		return i+1
	end
$$;
```

* supported languages
	* Java
	* Scala
	* Python
	* JavaScript
	* SQL
* function overloading supported
* if `CURRENT_SCHEMA` or `CURRENT_DATABASE` is called in UDF code, the UDF's (not session's) schema/database is returned  


## Stored procedures

### Example

```
CREATE OR REPLACE PROCEDURE myProc(from_table STRING, to_table STRING, count INT) {
	RETURNS STRING
	LANGUAGE PYTHON
	RUNTIME_VERSION = '3.12'
	PACKAGES = ('snowflake-snowpark-python')
	HANDLER = 'run'
as 
$$
def run(session, from_table, to_table, count) {
	session.table(from_table).limit(count).write.save_as_table(to_table)
}
$$;
```



