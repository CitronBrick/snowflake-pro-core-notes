


# Network Security


## Malicious IP Protection

continuously detects network & blocks access attempts from blacklisted IP addresses.

* ANONYMOUS_VPNS
* ANONYMOUS_PROXIES
* MALICIOUS_BEHAVIOR
* TOR_EXITS

In addition, Snowflake categorizes IP addresses into high-risk & low-risk labels.

```
Select * from LOGIN_HISTORY where IS_SUCCESS='NO' 
```

## MFA enforcement

being rolled out. Mandatory for all from Aug-Oct 2026

* For human users
	* passkey (based on **WebAuthn standard**) (less secure)
	* preferred Authenticator app
	* Duo (not replicated)

### MFA & SSO

By default, not required for SSO. 
To enforce MFA after IdP authentication, create a *Authentication Policy*

```
CREATE AUTHENTICATION POLICY ACCOUNTADMIN_DOUBLE_MFA
	AUTHENTICATION_METHODS = {'PASSWORD', 'SAML'}
	SECURITY_INTEGRATIONS = { '<SAML SECURITY INTEGRATIONS>' }
	MFA_ENROLLMENT = { 'REQUIRED' }
	MFA_POLICY = { ENFORCE_MFA_ON_EXTERNAL_AUTHENTICATION = 'ALL'}
```

We can restrict which MFA methods can be used

```
SHOW MFA METHODS FOR USER joe

ALTER USER JOE
```

Temporarily allow locked out user w/o MFA (in minutes)

`ALTER USER joe SET MINS_TO_BYPASS_MFA = 30` 


## Federated Authentication

* Service Provider (SP) : Snowflake
* Identity Provider (IdP) : 
	* External independant entity
		* creates & maintains user credentials & profile info
		* authenticates users for SSO
	* Native support provided by
		* Okta
		* Microsoft AD FS
	* Other SAML 2.0 compliant IdP
		* Google G-suite
		* Microsoft Entra ID
		* OneLogin
		* Ping Identity PingOne
* Enables
	* login
	* logout
	* inactivity -> timeout
* Multiple Idp usage is supported by the following drivers
	* JDBC
	* ODBC
	* Python

### Login
	
#### Snowflake initiated login

1. User comes to Snowflake UI
2. chooses IdP 
3. Users authenticates with IdP
4. IdP sends SAML response to Snowflake
5. Snowflake initiates session

#### IdP initiates login

1. User goes to IdP
2. logs into IdP
3. User selects Snowflake application
4. IdP sends SAML response to Snowflake
5. Snowflake initiates session
