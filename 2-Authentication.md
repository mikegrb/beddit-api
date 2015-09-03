# Authorization and authentication

Beddit API utilizes the OAuth2 as authentication protocol. In addition to
standard OAuth2 username/password authentication, backend provides the ability
to authenticate as an application. Authentication information must be sent to
backend within every request, in the Authorization HTTP header.

You must never store Beddit user's username or password in your application.
Instead, you exchange them to the access token, which is used to authenticate
all requests.

## Token types

Backend supports two token types: application and user tokens.

### Application tokens

Application authentication is used for requests that do not require user
authentication, for example creating a new user. Token is provided in the
HTTP authorization header in following format:

```
Authorization: Application <application id>:<application secret>
```

### User tokens

The user token is granted by backend to particular user using the flow described
below. Token is provided in the authorization header in following format:

```
Authorization: UserToken <user token>
```

## User authentication: Mobile flow

This flow is used in a case where the end user has trusted her password with the
client app. It conforms to OAuth2 specification section
http://tools.ietf.org/html/rfc6749#section-4.3

### POST /api/v1/auth/authorize

x-www-form-urlencoded field | Value
----------|------
grant_type | password
username | _User's username, typically the email address_
password | _User's password_

**Authentication**

No authentication headers are required.

**Response**

```javascript
{
    "user" : 7,
    "access_token" : "2YotnFZFEjr1zCsicMWpAA",
    "token_type" : "user",
    "expires_in" : 3600
}
```

Property | Meaning
------|--------
user | Id of the associated user
access_token | The access token to be used in Authorization header of the requests
token_type | The token type, currently supported type is "user"
expires_in | *Optional* In how many seconds the token expires. If missing, the token is not set to expire.

If the token is already expired, or otherwise cancelled, HTTP 403 error is
returned. In this case, the client application should ask the user to
authenticate again.


**Errors**

Error identifier | HTTP Status | Description
-----------------|-------------|------------
invalid_credentials | 400 | The password does not match, suggest reset?


## User authentication: Web flow

The OAuth2 web flow is intended for web, desktop and other applications. Instead
of the client application asking for username and password directly, the user is
directed from your app to Beddit website, where they user may choose to
authorize the client application to access user's sleep and other data.

To protect our user's privacy, Beddit strongly encourages applications to use
the web flow instead of the mobile flow wherever possible.

### Step 0: Registering your app

To use the web authentication flow, you need to register your app with Beddit.
You can do this by sending the following information to support@beddit.com.

* Name of your application
* A short description
* Homepage
* The app author's Beddit username (email used to create Beddit account)
* One or more redirect URL:s (for example, localhost for development, and one for staging and production environment). Note that other than local host, these need to be HTTPS.

Please note that the redirect URL:s need to include the HTTP/HTTPS part, and
also the full path. It is not sufficient to match the domain part of the URL.

For development purposes HTTP is fine, but for production use we require the
use of secure HTTPS URL.

We will then contact you with the required information to use the API:

* client_id
* client_secret


### Step 1: GET /api/v1/auth/authorize_web

First thing to do is to direct user to this authorization resource. The
following GET parameters are required.

Parameter name | Value
---------------|------
client_id | Your application's client_id
redirect_uri | Your application's redirect_uri
response_type | Must be set to **code**

If the user is not already signed in to Beddit, Beddit website will ask user's
username and password. After that, user may choose to authorize your application
to access her data.


### Step 2: Beddit redirects back to your application

Beddit redirects back to your application's redirect_uri specified in previous
request. Beddit passes one of the following parameters to the redirect_uri,
depending on whether the user authorized your application.

Parameter name | Value
---------------|------
code | A secret code that can be exchanged to an access token
error | If the user did not authorize your app, error is set to **access_denied**


### Step 3: POST /api/v1/auth/authorize

If all went well, the user authorized your application, and you can now exchange
the one-time code for an access token. Make a POST request with following
parameters set.

Parameter name | Value
---------------|------
client_id | Your applications id
redirect_uri | Your applications redirect_uri
client_secret | Your client_secret
grant_type | Must be set to **authorization_code**
code | The one-time, secret code you received in previous step

In response, you will get the access token for the user

**Example response**

```javascript
{
  "access_token" : "jcaurhmfeaisuxfmsecfkjebsf"
}
```


## Checking access token information

### GET /api/v1/auth/token_info

Get information about the access token used in the request.

**Authentication**

Request must be authenticated by User specific token.

**Example Response**

```javascript
{
  "user" : 1231,
  "token_type": "user"
}
```

Please see previous section for explanations of the access token properties.
