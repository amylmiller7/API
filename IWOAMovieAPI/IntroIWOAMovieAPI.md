# Introduction to the IWOA Movie API

- [What is the IWOA Movie API?](#what)
- [Getting started](#getStarted)
- [Environments](#environ)
- [Authentication and authorization](#authenAuthor)
    - [API keys](#APIKey)
    - [Request tokens](#requestToken)
    - [Session IDs](#sessionID)
- [Failures](#failures)
- [Supported content type](#suppContentType)
- [Rate limits](#rateLimits)
- [Pagination](#pagination)
- [Request default timeout](#requestDefTimeOut)
- [Response codes](#responseCodes)
    - [HTTP status codes](#HTTPStatusCodes)
    - [Error codes](#errorCodes)
- [API updates](#APIUpdates)

## <a id="what"></a>What is the IWOA Movie API?
_Inside the World of Amy_ contains numerous movie reviews. The mock IWOA Movie API allows developers to access this database of movie reviews for use in their own applications. With the API, developers can allow their users to create and maintain lists of movies. For example, consumers of their applications could create, update, or delete lists of favorite movies, movies based on genre or release year, or flag movies that they want to watch. They could also add and remove movies from their different lists.

## <a id="getStarted"></a>Getting started

1. Sign up for account.
2. Register for an [API key](#APIKey).
    1. Log into your account. 
    2. Click **Generate API key**. 
    3. Follow the steps on the screen.
3. Find endpoints and methods to retrieve information for your integration. For more information, see the Swagger documentation.


| Method | Endpoint | Description |
| ------ | -------- | ----------- |
| GET | /authentication/tokens/new | Get request token |
| GET | /authentication/sessions/new | Get session ID |
| GET | /accounts | Get account information |
| GET | /accounts/{accountId}/lists | Get movie lists |
| GET | /movies | Get all movies |
| GET | /movies/{movieId} | Get movie details |
| GET | /movies/genres | Get movie genres |
| POST | /lists/list | Create movie list |
| GET | /lists/list | Create movie list |
| GET | /lists/{listId} | Get movie list |
| DELETE | /lists/{listId} | Delete movie list |
| POST | /lists/{listId}/addItem | Add movie to movie list |
| POST | /list/{listId}/removeItem | Remove movie from movie list |
| GET | /ratelimit | Get rate limit information |

## <a id="environ"></a>Environments
The API provides two environments.

- Development: https://www.insidetheworldofamy.com/v1/api
- Production: https://development.insidetheworldofamy.com/v1/api

The environment specifies where requests through the API should be directed. 

Each environment uses different [API keys](#APIKey). When you switch between development and production environments, be sure to update the API keys in your code.

## <a id="authenAuthor"></a>Authentication and authorization
Authentication determines a user's identity. Authorization determines what permissions an authenticated user has for a set of resources. Authentication identifies you and authorization determines what you are allowed to do.

Authentication for the API is divided between the application and users.

- Application authentication uses [API keys](#APIKey).
- User authentication uses [request tokens](#requestToken) and [session IDs](#sessionID).
**Note:** Each user must have a separate request token and session ID.

Users are only authorized to access movies and genres, and to access, update, or delete movie lists associated with their account. Users cannot access or modify movie lists not associated with their account.

### <a id="APIKey"></a>API keys
API keys are simple encrypted strings that identify applications that use the API. API keys are associated with application accounts and should be used with all API calls as a query parameter in the following format:

`GET /movies?apiKey=abcdef123456`

The API recognizes and authenticates the application using the API key.

**Note:** The environments each use their own API key. When you switch from development to production, be sure to update the API keys in your code.

#### Security
API keys are used together with HTTPS to provide basic security. 

API keys are preferred to account user names and passwords as methods of authentication. API keys are associated with accounts but are not tied to account user names and passwords. As a result, you can easily revoke and regenerate API keys without needing to change your account credentials. 

Treat your API keys with the same security that you would passwords. 

- Do not share API keys.
- Do not embed API keys directly in code.
- Periodically generate new API keys.

#### How to generate API keys
Generate an API key from your account dashboard.

1. Log into your account.
2. Click **Generate API key**.
3. Follow the steps on the screen.


### <a id="requestToken"></a>Request tokens
Request tokens are temporary tokens that the application generates and uses to ask users for permissiont to access their account. The token expires after 60 minutes if it is not used. 

#### How to generate request tokens
Generate a request token through an API call. See `GET /authentication/tokens/new` in the Swagger documentation.

### <a id="sessionID"></a>Session IDs
A session ID authenticates (identifies) the user and authorizes (allows) the user to access and modify data associated with the user account.

The use of a session ID prevents users from seeing and modifying data in other accounts, for example, deleting movie lists not associated with their own account. A session ID must be included on all API calls that access or modify data associated with a user account.

Treat session IDs with the same security that you would passwords.

#### How to generate session IDs

1. Generate a [request token](#requestToken).
2. Ask the user to authorize the request token.
Send the user the following URL: https://www.insidetheworldofamy.com/authenticate/{requestToken}?redirectTo=http://www.insidetheworldofamy.com/approved. After they approve the request token, they will be redirected to the approved page.  
3. Create a new session ID with the authorized request token. 
See `GET /authentication/sessions/new` in the Swagger documentation.


## <a id="failures"></a>Failures
All API requests must be made over HTTPS. If API requests are made over HTTP, they will fail. If API requests lack [authentication](#authenAuthor), they will also fail. API methods that require a particular HTTP method will return an error if not invoked using the correct method. See [Error codes](#errorCodes).

## <a id="suppContentType"></a>Supported content type
The IWOA Movie API supports JSON representation of resources. The API uses the JSON data format for requests and responses. All requests should include the HTTP header `Content-Type=application/json`.

## <a id="rateLimits"></a>Rate limits
API usage is rate limited to protect the movie database from abuse. Your application can make up to 5,000 requests per hour. Request limits are tied to the [API key](#APIKey) (application authentication).

You can check the returned HTTP headers of any API request to see your current rate limit status.

```json
HTTP/1.1 200 OK
Date: Mon, 04 June 2018 8:05:32 GMT
Status: 200 OK
X-RateLimit-Limit: 3702
X-RateLimit-Remaining: 1298
X-RateLimit-Reset: 1528099532
```

| Header name | Description |
| ----------- | ---------- |
| `X-RateLimit-Limit` | Maximum number of API requests allowed per hour. |
| `X-RateLimit-Remaining` | Number of API requests remaining in the current rate limit window. |
| `X-RateLimit-Reset` | Time when the current rate limit window resets in UTC epoch seconds. |

You can also check your rate limit by calling `GET /rateLimit`. Calling this endpoint does not count against your API request limits. See the Swagger documentation.

**Note:** To conserve the consumption of resources, when you reach your rate limit, the API does not return a 429 error. 

## <a id="pagination"></a>Pagination
By default, the API endpoints return 10 pages with 25 items on each page per API request. You can modify the results for `GET /accounts/{accountId}/lists` and `GET /movies` with the following query parameters:

| Query parameter | Description | Notes |
| --------------- | ----------- | ----- |
| page | Specific page of results to return. | |
| pageLimit | Number of pages of results to return. | For use with `GET /movies` only. |
| itemLimit | Number of items to return per page. | Default: 25. |
| offset | Number of items to skip before collecting the result set. | For use with `GET /movies` only. To return items starting at item 100, set offset to 99. |


**Example requests**

`GET /movies?page=5&itemLimit10`
Returns page 5 of the results set with 10 movie objects on the page.

`GET /movies?offset=99`
Returns results starting with the 100th movie object in the results set.


**Example response**

The following is an example response payload for the request `GET /movies?page=5&pageSize10`. The request returns page 5 of the results set with 10 items on the page.

```json
{
    "page": 5,
    "itemLimit": 10
}
```
For more information about pagination query parameters for `GET /accounts/{accountId}/lists` or `GET /movies`, see the Swagger documentation.


## <a id="requestDefTimeOut"></a>Request default timeout
By default, all requests time out after 30 seconds. Requests taking longer than 30 seconds return at HTTP status code of 504 and an error code and error message in the response body. See [HTTP status code](#HTTPStatusCodes) and [Error codes](#errorCodes).

## <a id="responseCodes"></a>Response codes
The IWOA Movie API uses conventional [HTTP status codes](#HTTPStatusCodes) to indicate the result of a request. The API maps these HTTP status codes to [error codes and messages](#errorCodes), which are returned in response bodies.


### <a id="HTTPStatusCodes"></a>HTTP status codes

HTTP status codes indicate the type of error. Codes in the 4xx range indicate a client-side error, such as a missing request parameter, API key, or session ID. Codes in the 5xx range indicate a server-side error. 

| Code | Text | Description |
| ---- | ----------- | ------- |
| 200 | OK | Successfully returned requested data, such as a request token, session ID, or all movies in the database. |
| 201 | OK | Successfully created or modified an object, for example, created a movie list, added a movie to a movie list, or removed a movie from a movie list. |
| 400 | Bad Request | The request was invalid or cannot be served due to missing or incorrect parameters. |
| 401 | Unauthorized | Authentication failed due to such things as an invalid or missing API key or an invalid, missing, or expired session ID. |
| 403 | Forbidden | You are not authorized to perform the requested action. You do not possess the correct permissions to access the resource that you are attempting to access, such as movie lists associated with an account that is not your own. |
| 404 | Not Found | The URI requested is invalid or the resource requested, such as a movie list, does not exist. |
| 500 | Internal Server Error | The server encountered an unexpected condition, which prevented it from fulfilling the request. This is usually a temporary error, such as a high load situation or an endpoint temporarily having issues. |
| 503 | Service Unavailable | The server is currently unavailable because it is overloaded or down for maintenance. |
| 504 | Gateway timeout | The request timed out. |


### <a id="errorCodes"></a>Error codes
The error codes and messages provide detailed information about why a request failed. They are returned as JSON objects in the response body.

| Code | HTTP status code | Message |
| ---- | ----------- | ------- |
| 20 | 400 | The API key is missing from or incorrect in the query parameters. An API key, which is associated with a user account and generated from the user's account dashboard, is required for all API calls. |
| 21 | 400 | Query parameters are missing or incorrect. This error does not include a missing or incorrect API key. |
| 22 | 400 | Path parameters are missing or incorrect. |
| 23 | 400 | A request body is missing, incomplete, or incorrect. |
| 30 | 401 | Authentication failed due to an invalid API key. Retrieve a new API key from your account dashboard. |
| 31 | 401 | Authentication failed due to an invalid or expired session ID. Generate a new session ID by calling `GET /authentication/sessions/new`. See the Swagger documentation. |
| 40 | 404 | The endpoint that you called could not be found. | 
| 41 | 404 | No account was found with the ID that you provided. |
| 42 | 404 | No movie was found with the ID that you provided. |
| 43 | 404 | No movie list was found with the ID that you provided. | 
| 50 | 500 | An unknown internal server error occurred. The server encountered an unexpected condition that prevented it from fulfilling the request. |
| 60 | 503 | The server is currently unavailable because it is overloaded or down for maintenance. |
| 70 | 504 | The request timed out. The server took too long to respond to a request. |

**Example response**
```json
{  
    "errorCode": 20,
    "errorMessage": "Authentication failed due to an invalid API key. Retrieve a new API key from your account dashboard."
}
```

## <a id="APIUpdates"></a>API updates
2018-06-04 Introduced initial version of API.


