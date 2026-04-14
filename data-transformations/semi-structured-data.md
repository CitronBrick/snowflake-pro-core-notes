# Work with Semi structured data


## Formats 

* JSON
* XML
* orc
* parquet
* Avro

## Snowflake data types for semi - structured data

| key value 		| `OBJECT` 		|
| array 			| `ARRAY`		|
| hierarchical data | `VARIANT`		|

hierarchical data
1. extract into separate columns 
	* explicitly
	* auto-detect
2. `VARIANT`
	* explicit structure
	* auto-detect


## Semi structured Array (different from Structured array)

* Each value in an array is a `VARIANT`
* dynamically sized (can grow)
* can contain both JSON nulls & SQL NULLs
* created by `ARRAY_CONSTRUCT(exp1,exp2)` or `ARRAY_CONSTRUCT_COMPACT(expr1,expr2)` (skips null values)
* 
	```
		INSERT INTO tab1(col1) 
			SELECT ARRAY_CONSTRUCT('red', NULL,'blue')
	```

	```
		INSERT INTO tab1(col1)
			SELECT [12, NULL, 'fifteen'];
	```
* Nested arrays are possible `select arr[0][0]`
* `ARRAY_SLICE(arr, a, b)` : select [a,b[
* Max theoretical : size 128 Mb (practically < 128MB due to object overhead)
* `ARRAY_INSERT(arr, position, elt)`
* `ARRAY_APPEND(arr, elt)`
* `ARRAY_PREPREND(arr, elt)`

## Semi structured Object

* `VARCHAR` keys & `VARIANT` values
* key shouldn't be an empty string
* neither key nor value can be `NULL`
* Max value size : 128 Mb 

```
INSERT INTO tab2(col2)
	SELECT OBJECT_CONSTRUCT(
		'name', 'Jones'::VARIANT,
		'age', 42::VARIANT
	);

INSERT INTO tab2(col2)
	SELECT 
		{ 'name': 'Jones'::VARIANT, 'age' : 15::VARIANT};
```

### Object literals

* empty object : `{}`
* `select obj_col['key'] from tab`


## VARIANT

* can store values of *any* type including Object & Array.
* max 128 MB uncompressed data (practically less due to internal overhead)


```
CREATE OR REPLACE TABLE t1 (v  VARIANT);

INSERT INTO t1(v) 
	SELECT PARSE_JSON('{"key1": "value1", "key2": "value:2"}');
```


### Cast to Variant

* `TO_VARIANT(5)`
* `5::VARIANT`
* `CAST(5 AS VARIANT)`

* Variant with underlying type of `DATE`,`TIME`,`VARCHAR`,`TIMESTAMP`  will have value shown with **double quotes**
* Convert back to underlying type to remove double quotes : 'Exam'::VARIANT::VARCHAR
* Variant null (aka JSON null) is **different from** SQL null
* Variant null is a true value which compares equal to itself
* Variant null may be specified as "null"

### Extracting semi structured data into VARIANT

* When semi-structured data is loaded to VARIANT column, Snowflake uses rules to extract as much data as possible to columns.
* Unextracted data is stored in a single column in a parsed semi-structured structure.
* max 200 elements extracted per partition. For more contact Snowflake Support.
* following are not extracted
	* elements with even a single "null" value
	* elements that comtain multiple data types. Eg: `{"foo": 1}` & `{"foo": "1"}`
* if element was extracted to column, engine scans only column
* else engine scans entire structure -> *Performance impact*
* To avoid above
	* ensure each unique element stores values of a single data type
	* extract data elements containing "null" values to relational columns **before** laoding them 

## FLATTEN

* explodes compound values to multiple rows
* `FLATTEN` : table function that takes OBJECT/VARIANT/ARRAY column & produces *lateral/inline* view containing correlations  to other  tables that preceed it in the `FROM` clause.

### FLATTEN Input

1. `INPUT` : expression (mandatory)
2. `PATH` : path to element to be flattened. Default empty path => outermost element
3. `OUTER` : 
	* `TRUE` : exactly 1 row is generated for 0 row expansions (`NULL` in KEY/INDEX/VALUE columns)
	* `FALSE` (default) : 0 rows & inaccessible paths produce 0 rows
4. `RECURSIVE`
	* `TRUE` : expansion performed for all sub-elements recursively
	* `FALSE` (default)  : only elements referenced by `PATH` are expanded
5. `MODE` 	:  `OBJECT`,`ARRAY`, `BOTH` (Default)



### FLATTEN Output

1. `SEQ` : unique sequence number
2. `KEY` : for objects/maps
3. `PATH` : path to element that needs flattening
4. `INDEX` : element index if array, else `NULL`
5. `VALUE` : value of flattened object/array
6. `THIS` : element being flattened (after `PATH` arg is applied) (useful for recursion) 




Example 1: The below query ignores the empty (`NULL`) as `OUTER` is `FALSE` by default

```
SELECT * FROM TABLE(FLATTEN(INPUT => PARSE_JSON('[1, ,77]'))) f;

| SEQ 	| KEY 		| PATH 			| INDEX 	| VALUE 		| THIS 				|
|-----	|---------	|--------------	|----------	| -------------	| -----------------	|
| 1  	| NULL		| [0]			| 0			| 1				| [1,,77]			|
| 1  	| NULL		| [2]			| 2			| 77			| [1,,77]			|
```

Example 2: 

```
SELECT * FROM TABLE(FLATTEN(INPUT => PARSE_JSON('{"a":1, "b":[5,6]}', OUTER=> TRUE))) f;

| SEQ 	| KEY 		| PATH 			| INDEX 	| VALUE 		| THIS 				|
|-----	|---------	|--------------	|----------	| -------------	| -----------------	|
| 1  	| b			| b 			| NULL		| [5,6] 		| {"a":1, "b":[5,6]}|

SELECT * FROM TABLE(FLATTEN(INPUT => PARSE_JSON('{"a":1, "b":[5,6]}', PATH=> 'b'))) f;

| SEQ 	| KEY 		| PATH 			| INDEX 	| VALUE 		| THIS 				|
|-----	|---------	|--------------	|----------	| -------------	| -----------------	|
| 1  	| NULL		| b[0] 			| 0			| 5 			| [5,6]				|
| 1  	| NULL		| b[1] 			| 1			| 6 			| [5,6]				|

```

Exampl 3: `RECURSIVE`

explodes both inner objects & arrays

```
SELECT * FROM TABLE(FLATTEN(INPUT=>PARSE_JSON('{"a": 1, "b": [77,88], "c": {"d":"X"}}'))) f;

| SEQ 	| KEY 		| PATH 			| INDEX 	| VALUE 		| THIS 														|
|-----	|---------	|--------------	|----------	| -------------	| -----------------											|
| 1  	| a			| a 			| NULL		| 1 			| {"a": 1, "b": [77,88], "c": {"d":"X"}}					|
| 1  	| b			| b 			| NULL		| [77,88]		| {"a": 1, "b": [77,88], "c": {"d":"X"}}					|
| 1  	| c			| c 			| NULL		| {"d","X"} 	| {"a": 1, "b": [77,88], "c": {"d":"X"}}					|


SELECT * FROM TABLE(FLATTEN(INPUT=>PARSE_JSON('{"a": 1, "b": [77,88], "c": {"d":"X"}}'), RECURSIVE=>TRUE)) f;

| SEQ 	| KEY 		| PATH 			| INDEX 	| VALUE 		| THIS 														|
|-----	|---------	|--------------	|----------	| -------------	| -----------------											|
| 1  	| a			| a 			| NULL		| 1 			| {"a": 1, "b": [77,88], "c": {"d":"X"}}					|
| 1  	| b			| b 			| NULL		| [77,88]		| {"a": 1, "b": [77,88], "c": {"d":"X"}}					|
| 1  	| NULL		| b[0] 			| 0			| 77			| [77,88]													|
| 1  	| NULL		| b[1] 			| 1			| 88			| [77,88]													|
| 1  	| c			| c 			| NULL		| {"d":"X"} 	| {"a": 1, "b": [77,88], "c": {"d":"X"}}					|
| 1  	| d			| c.d 			| NULL		| X 			| {"d":"X"}													|

```

Example 4: MODE 

`MODE=OBJECT` explodes only object not array

``
SELECT * FROM TABLE(FLATTEN(INPUT=>PARSE_JSON('{"a": 1, "b": [77,88], "c": {"d":"X"}}'), RECURSIVE=>TRUE, MODE=`OBJECT`)) f;
| SEQ 	| KEY 		| PATH 			| INDEX 	| VALUE 		| THIS 														|
|-----	|---------	|--------------	|----------	| -------------	| -----------------											|
| 1  	| a			| a 			| NULL		| 1 			| {"a": 1, "b": [77,88], "c": {"d":"X"}}					|
| 1  	| b			| b 			| NULL		| [77,88]		| {"a": 1, "b": [77,88], "c": {"d":"X"}}					|
| 1  	| c			| c 			| NULL		| {"d":"X"} 	| {"a": 1, "b": [77,88], "c": {"d":"X"}}					|
| 1  	| d			| c.d 			| NULL		| X 			| {"d":"X"}													|
```


## LATERAL FLATTEN

Needed to use the result of a flatten in a different flatten call.

```
CREATE OR REPLACE TABLE persons AS
	SELECT column1 as id, PARSE_JSON(column2) as c
		FROM values (
			12712555,
			{
				name: {"first": "John", "last": "Smith"},
				contact: [
					{
						business: [
							{ type: "phone", content: "555-1234" },
							{ type: "email", content: "j.smith@example.com" }
						]
					}
				]
			},
		), (	
			98217771,
			'{ 
				name: { first: "Jane", last: "Doe" },
				contact: [
					{
						business: [
							{ type: "phone", content: "555-1236" },
							{ type: "email", content: "j.doe@example.com" }
						]
					}
				]
			}'
		);


SELECT id as "ID",
	f.value as "Contact",
	f1.value:type as "Type",
	f1.value:content as "Details"
FROM persons p,
	LATERAL FLATTEN(INPUT => p.c, PATH => 'contact' ) f,
	LATERAL FLATTEN(INPUT => f.value:business) f1;

+----------+-----------------------------------------+---------+-----------------------+
|       ID | Contact                                 | Type    | Details               |
|----------+-----------------------------------------+---------+-----------------------|
| 12712555 | {                                       | "phone" | "555-1234"            |
|          |   "business": [                         |         |                       |
|          |     {                                   |         |                       |
|          |       "content": "555-1234",            |         |                       |
|          |       "type": "phone"                   |         |                       |
|          |     },                                  |         |                       |
|          |     {                                   |         |                       |
|          |       "content": "j.smith@example.com", |         |                       |
|          |       "type": "email"                   |         |                       |
|          |     }                                   |         |                       |
|          |   ]                                     |         |                       |
|          | }                                       |         |                       |
| 12712555 | {                                       | "email" | "j.smith@example.com" |
|          |   "business": [                         |         |                       |
|          |     {                                   |         |                       |
|          |       "content": "555-1234",            |         |                       |
|          |       "type": "phone"                   |         |                       |
|          |     },                                  |         |                       |
|          |     {                                   |         |                       |
|          |       "content": "j.smith@example.com", |         |                       |
|          |       "type": "email"                   |         |                       |
|          |     }                                   |         |                       |
|          |   ]                                     |         |                       |
|          | }                                       |         |                       |
| 98127771 | {                                       | "phone" | "555-1236"            |
|          |   "business": [                         |         |                       |
|          |     {                                   |         |                       |
|          |       "content": "555-1236",            |         |                       |
|          |       "type": "phone"                   |         |                       |
|          |     },                                  |         |                       |
|          |     {                                   |         |                       |
|          |       "content": "j.doe@example.com",   |         |                       |
|          |       "type": "email"                   |         |                       |
|          |     }                                   |         |                       |
|          |   ]                                     |         |                       |
|          | }                                       |         |                       |
| 98127771 | {                                       | "email" | "j.doe@example.com"   |
|          |   "business": [                         |         |                       |
|          |     {                                   |         |                       |
|          |       "content": "555-1236",            |         |                       |
|          |       "type": "phone"                   |         |                       |
|          |     },                                  |         |                       |
|          |     {                                   |         |                       |
|          |       "content": "j.doe@example.com",   |         |                       |
|          |       "type": "email"                   |         |                       |
|          |     }                                   |         |                       |
|          |   ]                                     |         |                       |
|          | }                                       |         |                       |
+----------+-----------------------------------------+---------+-----------------------+
```


## Type Predicates

`IS_ARRAY`,`IS_BINARY`, `IS_VARCHAR`, `IS_DATE`,`IS_DATE_VALUE`, `IS_NULL_VALUE` (JSON null),
`IS_OBJECT`

Each of the above functions accepts a variant & returns a boolean.