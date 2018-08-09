# Cherish Developer Guide

Table of Contents
1. What is Cherish?
2. Authentication
3. Using API
4. Appendix

# 1. What is Cherish?

The Cherish is Internet connected motion bed. You can remote control on mobile app.
For more details see [Cherish Homepage](http://www.cgagu.com).

A developer can query his bed list and control bed state.
First make sure you have your bed connected and installed.


# 2. Authentication

We uses OAuth 2.0's authorization code grant flow to issue access tokens on behalf of users. Your web or mobile app should redirect users to the following URL:

```markdown
https://cherish.auth.us-east-1.amazoncognito.com
```


## Prerequisite

You'll need to obtain parameters for authentication flow, contact us.
- client id
- client secret

Also required inform your callback url to us.
- redirect uri


## Step 1 - Sending users to authorize and/or install

The /oauth2/authorize endpoint signs the user in.

```markdown
GET /oauth2/authorize
```

The /oauth2/authorize endpoint only supports HTTPS GET. The user pool client typically makes this request through the system browser, which would typically be Custom Chrome Tab in Android and Safari View Control in iOS.

The following values should be passed as GET parameters:

- response_type (Required): The response type. Must be code or token. Indicates whether the client wants an authorization code (authorization code grant flow) for the end user or directly issues tokens for end user (implicit flow).
- client_id (Required): The Client ID. Must be a pre-registered client in the user pool and must be enabled for federation.
- redirect_uri (Required): The URL to which the authentication server redirects the browser after authorization has been granted by the user. Must have been pre-registered with a client.
- state (Optional but strongly recommended): An opaque value the clients adds to the initial request. The authorization server includes this value when redirecting back to the client. This value must be used by the client to prevent CSRF attacks.


### Sample request

Authorization Code Grant Request

```markdown
GET https://cherish.auth.us-east-1.amazoncognito.com/oauth2/authorize?
response_type=code&
client_id=CLIENT_ID&
redirect_uri=REDIRECT_URI&
state=STATE
```


## Step 2 - Users are redirected to your server with a authorization code

If the user authorizes your app, We will redirect back to your specified redirect_uri with a temporary code in a code GET parameter, as well as a state parameter if you provided one in the previous step. If the states don't match, the request may have been created by a third party and you should abort the process.


### Sample response of positive requests

The authentication server redirects back to your app with the authorization code and state. The code and state must be returned in the query string parameters and not in the fragment. A query string is the part of a web request that appears after a '?' character; the string can contain one or more parameters separated by '&' characters. A fragment is the part of a web request that appears after a '#' character to specify a subsection of a document.

```markdown
HTTP/1.1 302 Found
Location: https://REDIRECT_URI?code=AUTHORIZATION_CODE&state=STATE
```


### Sample response of negative requests

The following are examples of negative requests:

If client_id and redirect_uri are valid but there are other problems with the request parameters (for example, if response_type is not included; if code_challenge is supplied but code_challenge_method is not supplied; or if code_challenge_method is not 'S256'), the authentication server redirects the error to client's redirect_uri.

```markdown
HTTP 1.1 302 Found Location: https://REDIRECT_URI?error=invalid_request
```

If the client requests 'code' or 'token' in response_type but does not have permission for these requests, the Amazon Cognito authorization server should return unauthorized_client to client's redirect_uri, as follows:
```markdown
HTTP 1.1 302 Found Location: https://REDIRECT_URI?error=unauthorized_client
```

If the client requests invalid, unknown, malformed scope, the Amazon Cognito authorization server should return invalid_scope to the client's redirect_uri, as follows:
```markdown
HTTP 1.1 302 Found Location: https://REDIRECT_URI?error=invalid_scope
```

If there is any unexpected error in the server, the authentication server should return server_error to client's redirect_uri. It should not be the HTTP 500 error displayed to the end user in the browser, because this error doesn't get sent to the client. The following error should return:
```markdown
HTTP 1.1 302 Found Location: https://REDIRECT_URI?error=server_error
```


## Step 3 - Exchanging a authorization code for an access token

The /oauth2/token endpoint gets the user's tokens.

```markdown
POST /oauth2/token
```

The /oauth2/token endpoint only supports HTTPS POST. The user pool client makes requests to this endpoint directly and not through the system browser.

Request Parameters in Header

- Authorization: If the client was issued a secret, the client must pass its client_id and client_secret in the authorization header through Basic HTTP authorization. The secret is Basic Base64Encode(client_id:client_secret).
- Content-Type: Must always be 'application/x-www-form-urlencoded'.

Request Parameters in Body

- grant_type (Required): Grant type. Must be authorization_code or refresh_token
- client_id (Required): Client ID. Must be a preregistered client in the user pool. The client must be enabled for Amazon Cognito federation. 
- redirect_uri (Required only if grant_type is authorization_code): Must be the same redirect_uri that was used to get authorization_code in /oauth2/authorize.
- code (Required if grant_type is authorization_code): The authorization code.
- refresh_token (Required if grant_type is refresh_token): The refresh token.


### Example of exchanging authorization code for tokens

Sample Request

```markdown
POST https://cherish.auth.us-east-1.amazoncognito.com/oauth2/token >
Content-Type='application/x-www-form-urlencoded'&
Authorization=Basic ENCODED_CLIENT_ID_AND_CLIENT_SECRET

grant_type=authorization_code&
client_id=CLIENT_ID&
code=AUTHORIZATION_CODE&
redirect_uri=https://REDIRECT_URI
```

Sample Response

```markdown
HTTP/1.1 200 OK
Content-Type: application/json

{
    "access_token": "eyJraWQiOiI5ZmRld1...xIHLysTMeXmYwcg",
    "refresh_token": "eyJjdHkiOiJKV1QiLC...oE2mjVF2OBmIw",
    "expires_in": 3600,
    "token_type": "Bearer"
}
```

Sample Error Response

```markdown
HTTP/1.1 400 Bad Request
Content-Type: application/json;charset=UTF-8

{
    "error":"invalid_request|invalid_client|invalid_grant|unauthorized_client|unsupported_grant_type|"
}
```

- invalid_request: The request is missing a required parameter, includes an unsupported parameter value (other than unsupported_grant_type), or is otherwise malformed. For example, grant_type is refresh_token but refresh_token is not included.
- invalid_client: Client authentication failed. For example, when the client includes client_id and client_secret in the authorization header, but there's no such client with that client_id and client_secret.
- invalid_grant: Refresh token has been revoked. Authorization code has been consumed already or does not exist.
- unauthorized_client: Client is not allowed for code grant flow or for refreshing tokens.
- unsupported_grant_type: Returned if grant_type is anything other than authorization_code or refresh_token.


### Example of exchanging refresh token for tokens

Sample Request

```markdown
POST https://cherish.auth.us-east-1.amazoncognito.com/oauth2/token >
Content-Type='application/x-www-form-urlencoded'
Authorization=Basic ENCODED_CLIENT_ID_AND_CLIENT_SECRET

grant_type=refresh_token&
client_id=CLIENT_ID&
refresh_token=REFRESH_TOKEN
```

Sample Response

```markdown
HTTP/1.1 200 OK
Content-Type: application/json

{
    "access_token": "eyJraWQiOiI5ZmRld1l...RUzkaBdmDTsxwing",
    "expires_in": 3600,
    "token_type": "Bearer"
}
```


## Step 4 - Using access tokens

The tokens awarded to your app can be used in requests to the API.

```markdown
https://api.onescape.io/SERVICE_NAME
```

The best way to communicate your access tokens, also known as bearer tokens, is by presenting them in a request's Authorization HTTP header:

```markdown
GET /RESOURCE_NAME
Authorization: Bearer ACCESS_TOKEN
```

This approach is required when using application/json with a write method.


# 3. Using API

The Cherish API is an interface for querying information from and enacting change in a bed.


## Functions

- Get information for all bed owned by the user
- Set control state of bed

### Supported bed control state

| Bed type | Actions (Enum) | Events | Conditions |
| --- | --- | --- | --- |
| motion | {headUp, headDown, allUp, allDown, footUp, footDown, stop, zg, flat, quiteSleep, memoryA, memoryB} | N/A | N/A |
| twin | {leftHeadUp, leftHeadDown, rightHeadUp, rightHeadDown, footUp, footDown, stop, zg, flat, quiteSleep, memoryA, memoryB} | N/A | N/A |

### Bed's GUI preview

| Type | Screenshot |
| --- | --- |
| motion | ![](type_motion.png) |
| twin | ![](type_twin.png) |


## Getting Started

First make sure your bed is connected to Internet and is functioning properly. Test that the smartphone app can control the bed.
Then you need to authentication to get access token and use it to control bed.

### YAML

You can use the YAML at [https://swaggerhub.com/apis/onescape/cherish](https://swaggerhub.com/apis/onescape/cherish)

### Auth sample screenshot

![](oauth_swaggerhub.jpeg)

![](oauth_postman.jpeg)


# 4. Appendix

If you have API key to authorize, you can use API key instead of access token. 


## Step 1 - Using key

The API key awarded to your app can be used in requests to the API.

```markdown
https://api.onescape.io/SERVICE_NAME
```

The best way to communicate your key is by presenting them in a request's X-Api-Key HTTP header:

```markdown
GET /RESOURCE_NAME
X-Api-Key: API_KEY
```

This approach is required when using application/json with a write method.


## Step 2 - Using API

Please refer to '3. Using API' for this step.

### Auth sample screenshot

![](apikey_swaggerhub.jpeg)

![](apikey_postman.jpeg)
