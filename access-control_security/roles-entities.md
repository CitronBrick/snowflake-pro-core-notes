# Roles & Entities

**Discretionary Access Control (DAC)** : object owner grants access to object

**Role Based Access Control (RBAC)** : privileges are assigned to roles, roles are asssigned to users

**User Based Access Control (UBAC)** : privileges are directly assigned to users. Considered only when `USE SECONDARY ROLE` is `ALL`

**Securable object** : an entity to which access is granted. Without grant access is denied

**Role** : an entity to which privilege is granted

**Privilege** : defined level of access to object. Granularity of access may be controlled by multiple privileges

**User** : user identity (person/service) recognized by Snowflake. Privileges can be granted to users


Organization: Account
Account: Warehouse, Database, Role, User, Other Account objects
Database: Database Role, Schema
Schema: Table, View, Stage, Stored Procedure, UDF, Other schema objects


## Roles

### System defined roles

* System defined roles cannot be dropped
* Privileges granted to System defined roles by Snowflake cannot be dropped

### GLOBALORGADMIN

* Organization Administration
* exists only in organization account
* view org level usage information

### ORGADMIN

* role that uses a regular account to manage operations at the organization level
* deprecated

### ACCOUNTADMIN

* encapsulates `SYSADMIN` & `SECURITYADMIN` roles
* should be granted to limited users

### SECURITYADMIN
* is granted `MANAGE GRANTS` privilege
* inherits `USERADMIN` role
* to create objects `SECURITYADMIN` also needs object creation privileges


#### MANAGE GRANTS

* privilege that allows to grant/revoke privileges
* not sufficient for `SECURITYADMIN` to create objects or other privileges


### USERADMIN

* is granted `CREATE USER` & `CREATE ROLE` privileges in the account


### SYSADMIN

* has privileges to create warehouses & databases
* recommended to assign all custom roles to SYSADMIN


### PUBLIC

* *pseudorole* granted to every user & role
* can own securable objects, except they are accessible to everyone by definition 