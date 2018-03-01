---
title: Alohomora API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - json

toc_footers:
  - <a href='http://alohomora.pw/'>Our website</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction
Welcome to Alohomora's API. Here is everything you need to know in order to build your own client.

# Keep in mind
```
X-ALOHOMORA-TOKEN: Your authentication token.
X-ALOHOMORA-SIGNATURE: The OpenPGP signature of the body of the request, signed with the private key associated to the public key you sent while authentication.
```

Every request made to the API should include the header "User-Agent" corresponding to your client's name.
This is the only one required until you have your authentication token.
At this point, you will have to include a request id called "req_id" in your request body and also thoses headers.

# Official structure for content
```json
{
	"type": 0,
	"name": "Name of the element which will be displayed"
	"customField": { "value": "valueofthefield", hidden: false },
	"customField": { "value": "value of the second field", hidden: true }
}

{
	"type": 0,
	"name": "Group name"
}
```
Even though this is the documentation for the API, I'd like to point out how "content" should be formatted.
This field is something you will see often during the usage of the API since it is the actual element that only the client can decrypt.
For the sake of clarity, we'll show you an example of how your client should encrypt and read them.

The first example is the content for an element, the second one is the content for a group.

More details on theses in the technical documentation.

# Registration
```json
{
	"id": 123,
	"token": "Generated token to be used directly"
}
```
> Returns:

> 201 - Created

> 400 - Missing fields or incorrect ones

> 409 - Username or email already taken

### Explanation
This enables you to let users register from your client.
This can be disabled from the server.

### HTTP Request
`PUT /users`

### Query Parameters
Parameter | Description
--------- | -----------
username  | User's username
email     | User's email
password  | User's password
machine_name | Machine name for the generated token
publickey | OpenPGP public key to be used for the generated token

# Authentication
## Challenge

```json
{
	"id": 123,
	"challenge": "the challenge that will be used when you authenticate"
}
```

Before you can authenticate, you need to have a challenge, that's a random string generated and stored on the server.
You need to store this until you are authenticated, then you can destroy it.

### HTTP Request
`GET /challenge`

## Logging in

You first need to get a token which is what you'll pass around all API calls.

```json
{
	"id": 123,
	"username": "user's name",
	"email": "user@email.tld",
	"isAdmin": false,
	"token": "token freshly generated on the server",
	"data" : {
		"group": [
			{
				"id": 123,
				"parent": 123, // Can be -1 for the root group
				"content": "Encrypted JSON containing a group"
			}
		],
		"element": [
			{
				"id": 12,
				"group": 123,
				"content": "Encrypted JSON containing an element"
			}
		],
		"config": [
			{
				"id": 1,
				"name": "The config name",
				"value": "The selected value"
			}
		]
	}
}
```
> Returns

> 200 - Logged in

> 403 - Bad username or password

### HTTP Request
`POST /users`

### Query Parameters
Parameter | Description
--------- | -----------
passcode  | SHA512(SHA512(username) + challenge + SHA512(password))
challenge | ID of the challenge you got
publickey | OpenPGP key used to sign the messages
machine_name | Machine name for the token

<aside class="notice">
The client has basically 2 password. The first one is the one the user sets in order to encrypt his data.<br />
The second one is the one used to authenticate on the API. In the official client, this password is the SHA512 of the first one.<br />
The idea comes from <a href="https://protonmail.com/blog/encrypted_email_authentication/">Protonmail's answer</a> to this problem.
The one in the query parameters is to authenticate on the API.
</aside>


# Request ID
The Request ID is just a number that you will include inside your request so that every one you make is unique.
On the server-side, an interger is stored in the database and every time you make an API call, this number is checked then incremented to be sure that someone is not re-using an older request that might have been intercepted.
This request id is just a parameter inside the body of your request called "req_id" and containing the current call number.

<aside class="notice">
From now on, you need to have the request ID and both X-ALOHOMORA-TOKEN and X-ALOHOMORA-SIGNATURE headers set.
</aside>

# Elements & Groups requests
## Adding an element

> Returns the ID of the added element

```json
{
	"id": 123
}
```

> Returns

> 201 - Created

This route lets you add an element of any kind.

### HTTP Request

`POST /element`

### Query Parameters
Parameter  | Description
---------- | -----------
parent_grp | Parent group id, can be -1 for root one
content    | Encrypted content of the element

## Modifying an element

> Returns

> 200 - Successfully modified

> 404 - Can't find an element with this ID

Basically the same as adding, but with a different HTTP verb, and the ID of the element.

### HTTP Request

`PUT /element`

### Query Parameters
Parameter  | Description
---------- | -----------
id         | ID of the element modified
parent_grp | Parent group id, can be -1 for root one
content    | Encrypted content of the element

## Deleting an element
> Returns

> 404 - Can't find an element with this ID

> 410 - Successfully deleted

This one deletes an element WITHOUT ASKING CONFIRMATION, that's the client's role.

### HTTP Request

`DELETE /element`

### Query Parameters
Parameter  | Description
---------- | -----------
id         | ID of the element to be removed

## Adding a group

> Returns the ID of the added group

```json
{
	"id": 123
}
```

> Returns

> 201 - Created


This route lets you add groups.

### HTTP Request

`POST /group`

### Query Parameters
Parameter  | Description
---------- | -----------
parent_grp | Parent group id, can be -1 for root one
content    | Encrypted content of the group

## Modifying a group

> Returns

> 200 - Modified

> 404 - Can't find the group with the given ID

Basically the same as adding, but with a different HTTP verb, and the ID of the group.

### HTTP Request

`PUT /group`

### Query Parameters
Parameter  | Description
---------- | -----------
id         | ID of the group modified
parent_grp | Parent group id, can be -1 for root one
content    | Encrypted content of the group

## Deleting a group

> This returns the list of deleted elements which were in the removed group

```json
{
	"deleted": [
		1,
		2,
		3
	]
}
```

> Returns


> 404 - Can't find group with this ID

> 410 - Successfully deleted

This one deletes an group WITHOUT ASKING CONFIRMATION, that's the client's role.
This also deletes every elements in it.

### HTTP Request

`DELETE /group`

### Query Parameters
Parameter  | Description
---------- | -----------
id         | ID of the group to be removed

## Getting updates from the server

```json
{
	"group": [
			{
				"id": 123,
				"parent": 123, // Can be -1 for the root group
				"content": "Encrypted JSON containing a group"
			}
	],
	"element": [
			{
				"id": 12,
				"group": 123,
				"content": "Encrypted JSON containing an element"
			}
	],
	"config": [
			{
				"id": 1,
				"name": "The config name",
				"value": "The selected value"
			}
	]
}
```

> Returns

> 200 - Everything went fine

### Explanation
This endpoint gives you every changes made since your last update with this token.
The client has to check if the Group / elements was already created on the client and update it accordingly.

### HTTP Request

`GET /update`

<aside class="warning">
You must not do timed requests to get updates from the API. <br />
The correct way to do is either to have a refresh button so that the user does it when he desires, or the more reliable way which is connecting to the push TCP server which will tell you when you need to call this route.
</aside>

# User data
## List token

```json
{
	"tokens": [
		{
			"id": 123,
			"name": "Encrypted machine name",
			"IP": "IP address of the last API call with the token",
			"loginTS": 123456789, // Timestamp of when the token was granted
			"lastUpdateTS": 123456789 // Timestamp of the last API call
		}
	]
}
```

### Explanation
This will list the tokens that are in the database.
You will not get the actual token, but rather the list of machine names, IPs and last login time.

### HTTP Request
`GET /token`

## Edit user's informations

> Returns

> 200 - Modified

> 403 - Trying to edit someone else's profile

### Explanation
This allows you to edit your own profile.

### HTTP Request

`PATCH /users`

### Query Parameters

Parameter | Required | Description
--------- | -------- | -----------
username  | false    | Update the user's username
email     | false | Update the user's email
password  | false    | New user password, probably the SHA512 of the one the users want to have

## Delete account

> Returns

> 403 - Trying to delete someone else

> 410 - Successfully deleted

### HTTP Request
`DELETE /users`

## Revoke token

> Returns

> 200 - Successfully revoked

> 403 - Token does not belong to the user

> 404 - No token with this ID

### Explanation
This API call only revoke the token, which means you won't be able to log with it again.
Though this also keeps it in the database, in case the user want to see IP or last login time.

### HTTP Request
`PUT /token`

## Delete token

> Returns

> 403 - Token does not belong to the user

> 404 - No token for this ID

> 410 - Successfully deleted

### Explanation
Contrary to revoke, deleting the token will fully erase it from the database. Only do this if you are sure you don't want to keep the data.

### HTTP Request
`DELETE /token`

# Administration requests

## Listing users

```json
{
	"users": [
		{
			"id": 123,
			"username": "user's name",
			"email": "user's email",
			"isAdmin": false
		}
	]
}
```

> Returns

> 200 - Request has worked

> 403 - You are not an admin

### HTTP Request

`GET /users`

### Query Parameters

Parameter | Required | Description
--------- | -------- | -----------
limit     | false    | Used for pagination
offset    | false    | Used for pagination

## Editing users

> Returns

> 200 - Request has worked

> 403 - You are not an admin


### Explanation
This allows you to edit a user's profile.

### HTTP Request

`PATCH /users`

### Query Parameters

Parameter | Required | Description
--------- | -------- | -----------
id        | true     | User's ID
username  | false | Update the user's username
email     | false     | Update the user's email
isAdmin   | false | Whether the user is admin or not
password  | false    | New user password, probably the SHA512 of the one the users want to have


## Deleting users

> Returns

> 403 - You are not an admin

> 410 - Successfully deleted


### HTTP Request

`DELETE /users`

### Query Parameters

Parameter | Required | Description
--------- | -------- | -----------
id        | true     | User's ID

## Getting server config

```json
{
	"register_captcha": false, // Not implemented yet
	"limit_update": 30, // Limit the amount of /update request per minute
	"public_register": true, // People can register by themself
	"api_registering": true // People can be registered through the API (No need for the website)
}
```

> Returns

> 200 - Config updated

> 403 - Not an admin

### Explanation
Returns the global server configuration

### HTTP Request
`GET /config`

## Updating server config

> Returns

> 200 - Configuration set

> 403 - Not an admin

### Explanation
Sets global server configuration

### HTTP Request
`PUT /config`

### Query Parameters
Parameter | Description
--------- | -----------
register_captcha | false
limit_update | 30
public_register| false
api_registering | false
