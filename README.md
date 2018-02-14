# What is Cherish?

The Cherish is motion bed connected to Internet
- Remote controlled by smartphone
- 7-day Schedule

For more details see [Cherish Homepage](http://www.cgagu.com).

# API

The Cherish API is an interface for querying information from and enacting change in a Cherish bed.

## YAML

You can use the YAML at [https://swaggerhub.com/apis/onescape/cherish](https://swaggerhub.com/apis/onescape/cherish)

## Supported

| Supported actions | Supported events | Supported conditions |
| --- | --- | --- |
| Set bed's control state to {headUp, headDown, allUp, allDown, legUp, legDown, stop, zg, flat, quiteSleep, reset} | N/A | N/A |

## Functions

- Get information for all bed owned by the user
- Set control state of bed

## Model

### Bed control state

| Type | Enum |
| --- | --- |
| string | {headUp, headDown, allUp, allDown, legUp, legDown, stop, zg, flat, quiteSleep, reset} |

### Controller view type

| Type | Enum |
| --- | --- |
| string | {motion, twin} |

Definition

| Value | Action | Preview |
| --- | --- | --- |
| motion | {headUp, headDown, allUp, allDown, legUp, legDown, stop, zg, flat, quiteSleep, reset} | ![alt text](motion.png?raw=true) |
| twin | N/A | N/A |

## How-to : OAuth2

### Step 1 - Obtain the access tokens

You'll need to obtain the access token with [OAuth Document](https://onescape.github.io/oauth) 

### Step 2 - Using access tokens

The tokens awarded to your app can be used in requests to the API.

```markdown
https://api.onescape.io/cherish
```

The best way to communicate your access tokens, also known as bearer tokens, is by presenting them in a request's Authorization HTTP header:

```markdown
GET /RESOURCE_NAME
Authorization: Bearer ACCESS_TOKEN
```

This approach is required when using application/json with a write method.

### Sample screenshot

![alt text](https://github.com/onescape/oauth/blob/master/swaggerhub.jpeg?raw=true)

![alt text](https://github.com/onescape/oauth/blob/master/postman.jpeg?raw=true)

## How-to : API key

The API key awarded to your app can be used in requests to the API.

```markdown
https://api.onescape.io/cherish
```

The best way to communicate your API key, is by presenting them in a request's X-API-Key HTTP header:

```markdown
GET /RESOURCE_NAME
X-API-Key: API_KEY
```

This approach is required when using application/json with a write method.

### Sample screenshot

![alt text](https://github.com/onescape/solarzon/blob/master/swaggerhub.jpeg?raw=true)

![alt text](https://github.com/onescape/solarzon/blob/master/postman.jpeg?raw=true)
