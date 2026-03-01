# Unstructured data


## Directory tables 

* implicit object (not a separate database object) layered on (internal or external) stage 
* stores the stage's data files' metadata (similar to an external table)
* can be added to a stage during creation (`create stage`) or later (`alter stage`) : `DIRECTORY ENABLE`
* Uses
	* query list of all unstructured files in stage
	* create views of unstructured data
	* construct file processing pipeline (using Snowpark API or external functions)

## SQL file functions

enables to access files staged in cloud storage

`GET_STAGE_LOCATION( @<stage_name> )` : returns url of external or internal named stage
`GET_RELATIVE_PATH( @<stage_name>, <absolute_file_path)` : returns path *relative to stage location* using *absolute file path*
`GET_ABSOLUTE_PATH( @<stage_name>, <relative_file_path> )` : returns absolute path
`GET_PRESIGNED_URL( @<stage_name>, <relative_file_path>, <expiration_time> )` : returns presigned url even if file does not exists (url returns `NoKeyError`)
`BUILD_SCOPED_FILE_URL( @<stage_name>, <relative_file_path>, <use_privatelink_host_for_businesscritical> )` : return scoped file url
`BUILD_STAGED_FILE_URL( @<stage_name>, <relative_file_path> )` : return staged file url

`GET_PRESIGNED_URL` & `BUILD_SCOPED_FILE_URL` are non-deterministic


### File Data Type functions

#### Constructors


##### TO_FILE

constructs file object

```
TO_FILE ( <stage_name>, <relative_path> )
TO_FILE ( <file_url> )
TO_FILE ( <metadata> )
```

##### TRY_TO_FILE

like `to_file` but returns `NULL` instead of throwing error


```
TRY_TO_FILE ( <stage_name>, <relative_path> )
TRY_TO_FILE ( <file_url> )
TRY_TO_FILE ( <metadata> )
```

#### Accessors

`FL_GET_CONTENT_TYPE`, `FL_GET_ETAG`, `FL_GET_FILE_TYPE`, `FL_GET_LAST_MODIFIED`, `FL_GET_RELATIVE_PATH`, `FL_GET_SCOPED_FILE_URL`, `FL_GET_SIZE` (BYTES), `FL_GET_STAGE`, `FL_GET_STAGE_FILE_URL` 

`FL_IS_AUDIO`, `FL_IS_COMPRESSED`, `FL_IS_DOCUMENT`, `FL_IS_IMAGE`, `FL_IS_VIDEO` 

The above functions accept either of the following arguments:

##### file object 

```
CREATE TABLE t1 (f FILE);
INSERT INTO t1 SELECT TO_FILE(BUILD_STAGE_FILE_URL('@mystage', 'image.png'));

SELECT FL_GET_STAGE_FILE_URL(f) from t1
```

##### Variant object

```
CREATE TABLE t1 (f OBJECT);
INSERT INTO t1 (
	SELECT OBJECT_CONSTRUCT(
		'STAGE_FILE_URL', 'https://snowflake.account.snowflakecomputing.com/api/files/TEST/PUBLIC/MYSTAGE/image.png',
		'ETAG', '<ETAG value>', 'LAST_MODIFIED', 'Wed, 11 Dec 2024 20:24:00 GMT', 
		'SIZE', 105859, 'CONTENT_TYPE', 'image/jpg'
	)
);

SELECT FL_GET_STAGE_FILE_URL(f) from t1;
```

## Data File URL types

### Presigned url

* Navigate via presigned url directly into browser
* Retrieve a presigned url in Snowsight. Click on the presigned url in the results table
* Send presigned url in a request to REST API for file support
* default 1 hour (3600).
* Max 7 days 
* For Microsoft Fabric OneLake max 1 hour

### Scoped url 

* encoded
* allows access to specific file for limited time.
* valid until result cache expires (24h)

### Stage file url

* does not expire
* call it in query, UDF or stored procedure
* send to REST API, for file access (authenticate, authorize, redirect to file)