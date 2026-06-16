


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
* Enables SSO
	* login
	* logout
	* inactivity -> timeout
* Multiple Idp usage is supported by the following drivers
	* JDBC
	* ODBC
	* Python

### Login with SSO
	
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

### Logout

#### Standard Logout

* requires users to explicitly logout of IdP **and** Snowflake
* all IdPs support standard logout

#### Global Logout

* logout of IdP & all its Snowflake session
* IdP dependant support


## Key pair authentication

* min length of RSA pair: 2048
* encrypted private keys are better than unencyrpted private keys
* Key generation algos
	* RSA algos
		* RS256
		* RS384
		* RS512
	* Elliptic Curve Digital Signature Algorithms (ECDSA)
		* ES256
		* ES384
		* ES512
* Hash algos
	* SHA-256
	* SHA-384
	* SHA-512
1. Generate private key
	* `openssl genrsa 2048 | openssl pkcs8 -topk8 -inform PEM -out rsa_key.p8`
	* add `-nocrypt* to above command to generate .pem without encryption`
2. Generate public key
	* `openssl rsa -in rsa_key.p8 -pubout -out rsa_key.pub`
3. Store private & public keys securely, using file permissions

## Grant public key to a user

* The following privileges are needed to assign a public key to a user
	* `MODIFY PROGRAMMATIC AUTHENTICATION METHODS` on user
	* `OWNERSHIP` on user
* `GRAMT MODIFY PROGRAMMATIC AUTHENTICATION METHODS ON USER my_service_user TO ROLE my_service_owner_role`

* `ALTER USER SET RSA_PUBLIC_KEY='slfjwoei';`

## Verify user's public key's finger print

```
DESC USER example_user
	--> SELECT SUBSTR(
		(SELECT "value" from $1
			WHERE "property" = 'RSA_PUBLIC_KEY_FP'), 
		LEN('SHA256:') + 1) AS key;
```

```
openssl rsa -pubin rsa_key.pub -outform DER | openssl dgst -sha256 -binary | openssl enc -base64
```
The outputs of the above 2 commands should match.
```