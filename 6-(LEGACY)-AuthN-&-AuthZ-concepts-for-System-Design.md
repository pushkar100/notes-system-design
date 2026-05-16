# AuthN and AuthZ concepts for System Design

## Table of Contents

- [AuthN and AuthZ concepts for System Design](#authn-and-authz-concepts-for-system-design)
  - [Authentication vs Authorization](#authentication-vs-authorization)
  - [Cookie-based or stateful authentication](#cookie-based-or-stateful-authentication)
  - [Token-Based or Stateless Authentication using JWTs](#token-based-or-stateless-authentication-using-jwts)
  - [Third party Auth mechanisms with OAuth and OpenID](#third-party-auth-mechanisms-with-oauth-and-openid)
    - [OAuth for 3rd party authorization](#oauth-for-3rd-party-authorization)
    - [OpenID Connect or OIDC for 3rd party authentication](#openid-connect-or-oidc-for-3rd-party-authentication)
  - [SAML for Enterprise SSO](#saml-for-enterprise-sso)
  - [JWT structure](#jwt-structure)
  - [Summary](#summary)

## Authentication vs Authorization

- **Authentication (AuthN)**: Verifying ***who you are***. (e.g., Checking ID at the airport gate)
- **Authorization (AuthZ)**: Verifying ***what you can do***. (e.g., Checking if your ticket allows access to the First Class lounge)

## Cookie-based or stateful authentication

**Concept**: The *server creates a "Session" in its memory/database* and *gives the client a reference ID (Session ID) via a Cookie*

Flow:
1. User logs in i.e the attempt is successful (Ex: username and password checks passed)
2. Server creates a session record `{'id': '123', 'user': 'Alice'}` in Redis/DB
3. Server sends `Set-Cookie: session_id=123` (**`Set-Cookie`** is a response header set by a server)
4. Browser automatically sends this cookie on every request (Once the client has received and stored a cookie, it will send a **`Cookie`** header in *subsequent requests to the same domain that originally set the cookie*, provided the cookie hasn't expired and other parameters like `Path` and `Secure` flags are met)

```
 [Browser]             [Server]              [Redis/DB]
   | (1) Login           |                      |
   |-------------------->| (2) Create Session   |
   |                     |--------------------->|
   | (3) Set-Cookie: ID  |                      |
   |<--------------------|                      |
   |                     |                      |
   | (4) GET /data       | (5) Check ID in DB   |
   | Cookie: ID          |--------------------->|
   |-------------------->|                      |
```

**Use Case**: ***Traditional monolithic web apps*** (Rails, Django), ***Banking*** (high security)

**Pros**: ***Easy to revoke*** (just delete from Redis), smaller payload size

**Cons**: ***Stateful***
1. *Harder to **scale horizontally***; requires *"Sticky Sessions"* or *centralized Redis*)
2. *Vulnerable to **CSRF***

## Token-Based or Stateless Authentication using JWTs

**Concept**: The *server generates a signed token (JWT) containing user data*. The *server does not save a session*. The *token is the session*

Flow:
1. User logs in i.e the attempt is successful (Ex: username and password checks passed)
2. Server creates a *JSON payload, signs it with a **secret key**, and sends it to Client*
3. Client saves it ("LocalStorage" or "Cookie")
4. Client sends token in `Header: Authorization: Bearer <token>`
5. *Server validates signature* (CPU operation, no DB lookup)

```
 [Browser]             [Server]
   | (1) Login           |
   |-------------------->| (2) Sign JWT (Secret Key)
   | (3) Return JWT      |
   |<--------------------|
   |                     |
   | (4) GET /data       | (5) Verify Signature (CPU)
   | Auth: Bearer JWT    |     (No DB Call needed)
   |-------------------->|
```

**Use Case**: ***Microservices, SPAs (React/Vue), Mobile Apps***

**Pros**: ***Stateless*** 
1. ***Infinitely scalable***
2. ***Works across different domains*** (CORS friendly).

**Cons**: **Hard to revoke** (cannot "delete" a valid signature without blacklisting or waiting for expiry), larger payload
- ***Solution***: The server *embeds an expiration timestamp* (the `exp` claim) directly into the token's payload. Upon receipt, the server simply compares the current time to this timestamp; if the current time is greater, the token is invalid. This check is **stateless** and requires *no* database lookup

## Third party Auth mechanisms with OAuth and OpenID

### OAuth for 3rd party authorization

**OAuth 2.0 (Authorization / Delegation)**
- **Concept**: A protocol that allows a user to *grant a third-party application access* to their resources *without sharing their password*. Think of it as giving a "Valet Key" that only starts the car but doesn't open the trunk

**Example**: "Log in with Google" or allowing an app to import your Gmail contacts

```
 [User]     [App (Client)]      [Google (Auth Server)]
  |              |                      |
  | (1) Click "Import Contacts"         |
  |------------->| (2) Redirect to Google
  |              |--------------------->|
  |                                     | (3) User logs in & Approves
  |              | (4) Auth Code        |
  |              |<---------------------|
  | (5) Send Code to Backend            |
  |------------->| (6) Exchange Code for Access Token
  |              |--------------------->|
  |              | (7) Access Token     |
  |              |<---------------------|
  |              | (8) Use Token to fetch Contacts
  |              |--------------------->|
```

**Use Case**: Third-party integrations, API Access

**Pros**: Secure delegation, passwords never shared

**Cons**: Complex flow to implement

**How it works**:
1. **Authorization Request**: The user clicks "Login" and gets redirected to the **Provider** (e.g., Google) with a request for specific permissions (scopes)
2. **User Consent**: The user logs in at the **Provider** and approves the permissions
3. **Authorization Code**: The **Provider** redirects the user back to your app with a ***temporary Authorization Code***.
4. **Token Exchange**: Your app's server sends this **`Code + your Client Secret`** to the Provider to *prove its identity*
	- *Security*: If an attacker gains access to the temporary code (because it is sent via the URL from provider to client), they still do not have the client secret (every client will have a predetermined client secret)! Hence, safe!
5. **Access Token**: The Provider validates the Code/Secret and *returns an Access Token (used to fetch data)*

```
+--------+                               +---------------+
|        |--(1) "Login" -> Redirect ---->|               |
|  User  |                               | Authorization |
| Browser|<-(2) User approves Access --->|    Server     |
|        |                               | (e.g. Google) |
|        |<-(3) Sends "Auth Code" -------|               |
+---|----+                               +-------|-------+
    |                                            |
    | (4) Send "Auth Code" + "Client Secret"     |
    +------------------------------------------->|
                                                 |
      (5) Returns "Access Token"                 |
    <--------------------------------------------+
    |
+---|----+
| Client |
| Server |
+--------+
```

**Note**: 
1. This access token is further sent to resource servers as an **authorization header i.e stateless , may be a JWT**
	- Ex: `Authorization: Bearer <your-access-token-string>` which needs to be validated by the resource server
2. The **Refresh Token** is a *special, long-lived credential* used to get new Access Tokens *without forcing the user to log in again*
	- **Where do we hide it? (Storage)**
		1. ***For Websites*** (***The Safe Way***): We store it in an **`HttpOnly`** Cookie
			- *Why?* This is a special cookie that your *website's JavaScript code cannot see*. This means if a hacker tricks your browser into running bad code (*XSS attack*), the hacker *cannot* steal the key because the browser refuses to show it to them
			- *Analogy*: It's like a wristband that is glued to your wrist. You can't take it off or hand it to anyone else; you can only show it to the bouncer
		2. ***For Mobile Apps***: Phones have a *built-in "digital safe" that is encrypted*
			- iOS: Keychain
			- Android: Keystore
			- *Analogy*: Putting the key inside a safe that requires the phone's password/FaceID to open

### OpenID Connect or OIDC for 3rd party authentication 

**Concept**: A *thin identity layer* on top of OAuth 2.0.
- OAuth handles "Access" (Here is a token to fetch photos)
- OIDC handles "**Identity**" (Here is an ID Token that says *who this user is*)

**Key Difference**: OAuth returns an Access Token. ***OIDC returns an ID Token (usually a JWT)***

**Use Case**: "Log in with X" (**SSO** for consumers)

**Pros**: ***Standardized way to get user profile info (email, name)***

|Feature                            |ID Token (OpenID Connect)|Access Token (OAuth 2.0)                                                 |
|-----------------------------------|-------------------------|-------------------------------------------------------------------------|
|Purpose                            |Authentication (Who you are)|Authorization (What you can do)                                          |
|Analogy                            |ID Badge / Passport      |Key Card / Ticket                                                        |
|Audience                           |Read by the Client App (Frontend)|Read by the API (Resource Server)                                        |
|Contents                           |User Profile (name, email, picture)|Permissions (scope="read:files"), Expiry                                 |
|Format                             |Always a JWT             |Can be JWT or Opaque (Random String)                                     |


## SAML for Enterprise SSO

**Concept**: An *older, XML-based standard* for exchanging auth data between an Identity Provider (IdP) and a Service Provider (SP).

- Flow: Similar to OAuth/OIDC but *uses heavy XML documents and browser redirects*
- Use Case: *Enterprise B2B* (Connecting corporate Active Directory to Salesforce, Jira, or AWS)
- Pros: The standard for corporate enterprise environments
- Cons: *XML is verbose and notoriously difficult to parse/debug*; aging technology

## JWT structure

JWT Structure (Separated by dots `.`)
1. **Header**: Algorithm type (e.g., `HS256`)
2. **Payload** (Claims): Data (`user_id: 123`, `exp: 1712000000`, `role: admin`)
3. **Signature**: `Hash(Header + Payload + SecretKey)`. This ensures *integrity*!

|Storage Method                     |Security Risk           |Mitigation                                                               |
|-----------------------------------|------------------------|-------------------------------------------------------------------------|
|LocalStorage                       |XSS: JS can read the token and send it to attacker.|Strict Content Security Policy (CSP), Sanitize inputs.                   |
|`HttpOnly` Cookie                    |CSRF: Browser auto-sends cookie to attacker's request.| `SameSite=Strict` attribute, Anti-CSRF Tokens.                             |

## Summary

|Mechanism                          |Primary Goal            |State?                                                                   |Best For                               |
|-----------------------------------|------------------------|-------------------------------------------------------------------------|---------------------------------------|
|Session/Cookie                     |AuthN                   |Stateful                                                                 |Monoliths, Banking, High Security reqs.|
|JWT                                |AuthN                   |Stateless                                                                |Microservices, High Scale, Mobile Apps.|
|OAuth 2.0                          |AuthZ (Access)          |Stateless                                                                |Granting API access to 3rd parties.    |
|OIDC                               |AuthN (Identity)        |Stateless                                                                |Consumer Single Sign-On (SSO).         |
|SAML                               |AuthN (Identity)        |Stateless                                                                |Enterprise/Corporate SSO (B2B).        |
