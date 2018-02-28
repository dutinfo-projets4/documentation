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

# Authentication
## Challenge

Before you can authenticate, you need to have a challenge, that's a random string generated and stored on the server.
You need to store this until you are authenticated, then you can destroy it.

```json
{
	"id": 123,
	"challenge": "the challenge that will be used when you authenticate"
}
```

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

### HTTP Request
`POST /users`

### Query Parameters
Parameter | Description
--------- | -----------
passcode  | SHA512(SHA512(username) + challenge + SHA512(password))
challenge | ID of the challenge you got
publickey | OpenPGP key used to sign the messages

<aside class="notice">
The client has basically 2 password. The first one is the one the user sets in order to encrypt his data.<br />
The second one is the one used to authenticate on the API. In the official client, this password is the SHA512 of the first one.<br />
The idea comes from <a href="https://protonmail.com/blog/encrypted_email_authentication/">Protonmail's answer</a> to this problem.
The one in the query parameters is to authenticate on the API.
</aside>


# Request ID
The Request ID is just a number that you will include inside your request so that every one you make is unique.
On the server-side, an interger is stored in the database and every time you make an API call, this number is checked to be sure that someone is not re-using an older request that might have been intercepted.
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
This route lets you add an element of any kind.

### HTTP Request

`POST /element`

### Query Parameters
Parameter  | Description
---------- | -----------
parent_grp | Parent group id, can be -1 for root one
content    | Encrypted content of the element

## Modifying an element

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

This one deletes an element WITHOUT ASKING CONFIRMATION, that's the client's role.

### HTTP Request

`DELETE /element`

### Query Parameters
Parameter  | Description
---------- | -----------
id         | ID of the element to be removed

## Adding an group

> Returns the ID of the added group
```json
{
	"id": 123
}
```
This route lets you add groups.

### HTTP Request

`POST /group`

### Query Parameters
Parameter  | Description
---------- | -----------
parent_grp | Parent group id, can be -1 for root one
content    | Encrypted content of the group

## Modifying an group

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

### Explanation
This endpoint gives you every changes made since your last update with this token.
The client has to check if the Group / elements was already created on the client and update it accordingly.

### HTTP Request

`GET /update`

<aside class="warning">
You may never do timed requests to get updates from the API. <br />
The correct way to do is either to have a refresh button so that the user does it when he desires, or the more reliable way which is connecting to the push TCP server which will tell you when you need to call this route.
</aside>

# User edition
## Edit user's informations
### Explanation
This allows you to edit your own profile.

### HTTP Request

`PATCH /users`

### Query Parameters

Parameter | Required | Description
--------- | -------- | -----------
username  | true     | Update the user's username
email     | true     | Update the user's email
password  | false    | New user password, probably the SHA512 of the one the users want to have

## Delete account
### HTTP Request
`DELETE /users`

# Configuration requests
## Setting config

```json
{
	"id": 123
}
```

### Explanation
These are server-side configuration. This is not encrypted, since no confidential element should be stored.
If your client adds custom config, their name should be prefixed by the name of your client.

### HTTP Request
`POST /config`

### Query Parameters
Parameter | Description
--------- | -----------
name      | Name of the config
value     | Value of the config

## Updating config
### HTTP Request
`PUT /config`

### Query Parameters
Parameter | Description
--------- | -----------
id        | ID of the config
name      | Name of the config
value     | Value of the config

## Deleting config
### Explanation
Should probably not be used. Deletes config from the server.

### HTTP Request
`DELETE /config`

### Query Parameters
Parameter | Description
--------- | -----------
id        | ID of the config


# Administration requests
