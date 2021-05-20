# API

## Users

### Create User
To start working with the Mainflux system, you need to create a user account.

> Must-have: e-mail and password (password must contain at least 8 characters)

```
curl -s -S -i -X POST -H "Content-Type: application/json" http://localhost/users -d '{"email":"<user_email>", "password":"<user_password>"}'
```

Response:
```
HTTP/1.1 201 Created
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 15:06:45 GMT
Content-Type: application/json
Content-Length: 0
Connection: keep-alive
Location: /users/d782b42b-e317-4cd7-9dd0-4e2ea0f349c8
Strict-Transport-Security: max-age=63072000; includeSubdomains
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: *
Access-Control-Allow-Headers: *
```

### Create Token
To log in to the Mainflux system, you need to create a `user_token`.

> Must-have: registered e-mail and password

```
curl -s -S -i -X POST -H "Content-Type: application/json" http://localhost/tokens -d '{"email":"<user_email>", 
"password":"<user_password>"}'
```

Response:
```
HTTP/1.1 201 Created
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 15:07:18 GMT
Content-Type: application/json
Content-Length: 281
Connection: keep-alive
Strict-Transport-Security: max-age=63072000; includeSubdomains
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: *
Access-Control-Allow-Headers: *

{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2MTU0MjQ4MzgsImlhdCI6MTYxNTM4ODgzOCwiaXNzIjoibWFpbmZsdXguYXV0aCIsInN1YiI6InRlc3RAZW1haWwuY29tIiwiaXNzdWVyX2lkIjoiZDc4MmI0MmItZTMxNy00Y2Q3LTlkZDAtNGUyZWEwZjM0OWM4IiwidHlwZSI6MH0.TAQxV6TImKw06RsK0J11rOHiWPvexEOA4BNZnhLhtxs"}
```

### Get User
You can always check the user entity that is logged in by entering the user ID and `user_token`.

> Must-have: `user_id` and `user_token`

```
curl -s -S -i -X GET -H "Authorization: <user_token>" http://localhost/users/<user_id>
```

Response:
```
HTTP/1.1 200 OK
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 15:09:47 GMT
Content-Type: application/json
Content-Length: 85
Connection: keep-alive
Strict-Transport-Security: max-age=63072000; includeSubdomains
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: *
Access-Control-Allow-Headers: *

{"id":"d782b42b-e317-4cd7-9dd0-4e2ea0f349c8","email":"test@email.com"}
```

### Get All Users
You can get all users in the database by calling the this function

> Must-have: `user_token`

```
curl -s -S -i -X GET -H "Authorization: <user_token>" http://localhost/users
```

Response:
```
HTTP/1.1 200 OK
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 15:11:28 GMT
Content-Type: application/json
Content-Length: 217
Connection: keep-alive
Strict-Transport-Security: max-age=63072000; includeSubdomains
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: *
Access-Control-Allow-Headers: *

{"total":2,"offset":0,"limit":10,"Users":[{"id":"4bf4a13a-e9c3-4207-aa11-fe569986c301","email":"admin@example.com"},{"id":"d782b42b-e317-4cd7-9dd0-4e2ea0f349c8","email":"test@email.com"}]}
```

### Update User
Updating user's metadata

> Must-have: `user_token`

```
curl -s -S -i -X PUT -H "Content-Type: application/json" -H "Authorization: <user_token>" http://localhost/users -d 
'{"metadata":{"foo":"bar"}}'
```

Response:
```
HTTP/1.1 200 OK
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 15:15:31 GMT
Content-Type: application/json
Content-Length: 0
Connection: keep-alive
Strict-Transport-Security: max-age=63072000; includeSubdomains
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: *
Access-Control-Allow-Headers: *
```

### Change Password
Changing the user password can be done by calling the update password function

> Must-have: `user_token`, `old_password` and password (`new_password`)

```
curl -s -S -i -X PATCH -H "Content-Type: application/json" -H "Authorization: <user_token>" http://localhost/password -d '{"old_password":"<old_password>", "password":"<new_password>"}'
```

Response:
```
HTTP/1.1 201 Created
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 15:17:36 GMT
Content-Type: application/json
Content-Length: 11
Connection: keep-alive
Strict-Transport-Security: max-age=63072000; includeSubdomains
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: *
Access-Control-Allow-Headers: *
```

## Things

### Create Thing
To create a thing, you need the thing and a `user_token`

> Must-have: `user_token`

```
curl -s -S -i -X POST -H "Content-Type: application/json" -H "Authorization: <user_token>" http://localhost/things/bulk -d '[{"name": "<thing_name>"}]'
```

Response:
```
HTTP/1.1 201 Created
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 15:18:37 GMT
Content-Type: application/json
Content-Length: 119
Connection: keep-alive
Access-Control-Expose-Headers: Location

{"things":[{"id":"64140f0b-6448-41cf-967e-1bbcc703c332","name":"thing_name","key":"659aa6ca-1781-4a69-9a20-689ddb235506"}]}
```

### Create Things
You can create multiple things at once by entering a series of things structures and a `user_token`

> Must-have: `user_token` and at least two things

```
curl -s -S -i -X POST -H "Content-Type: application/json" -H "Authorization: <user_token>" http://localhost/things/bulk -d '[{"name": "<thing_name_1>"}, {"name": "<thing_name_2>"}]'
```

Response:
```
HTTP/1.1 201 Created
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 15:19:48 GMT
Content-Type: application/json
Content-Length: 227
Connection: keep-alive
Access-Control-Expose-Headers: Location

{"things":[{"id":"4328f3e4-4c67-40b3-9491-0ab782c48d50","name":"thing_name_1","key":"828c6985-c2d6-419e-a124-ba99147b9920"},{"id":"38aa33fe-39e5-4ee3-97ba-4227cfac63f6","name":"thing_name_2","key":"f73e7342-06c1-499a-9584-35de495aa338"}]}
```

### Get Thing
You can get thing entity by entering the thing ID and `user_token`

> Must-have: `user_token` and `thing_id`

```
curl -s -S -i -X GET -H "Authorization: <user_token>" http://localhost/things/<thing_id>
```

Response:
```
HTTP/1.1 200 OK
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 15:20:52 GMT
Content-Type: application/json
Content-Length: 106
Connection: keep-alive
Access-Control-Expose-Headers: Location

{"id":"64140f0b-6448-41cf-967e-1bbcc703c332","name":"thing_name","key":"659aa6ca-1781-4a69-9a20-689ddb235506"}
```

### Get All Things
Get all things, list requests accepts limit and offset query parameters

> Must-have: `user_token`

```
curl -s -S -i -X GET -H "Authorization: <user_token>" http://localhost/things
```

Response:
```
HTTP/1.1 200 OK
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 15:21:49 GMT
Content-Type: application/json
Content-Length: 391
Connection: keep-alive
Access-Control-Expose-Headers: Location

{"total":3,"offset":0,"limit":10,"order":"","direction":"","things":[{"id":"64140f0b-6448-41cf-967e-1bbcc703c332","name":"thing_name","key":"659aa6ca-1781-4a69-9a20-689ddb235506"},{"id":"4328f3e4-4c67-40b3-9491-0ab782c48d50","name":"thing_name_1","key":"828c6985-c2d6-419e-a124-ba99147b9920"},{"id":"38aa33fe-39e5-4ee3-97ba-4227cfac63f6","name":"thing_name_2","key":"f73e7342-06c1-499a-9584-35de495aa338"}]}
```

### Update Thing
Updating a thing entity

> Must-have: `user_token` and `thing_id`

```
curl -s -S -i -X PUT -H "Content-Type: application/json" -H  "Authorization: <user_token>" http://localhost/things/<thing_id> -d '{"name": "<thing_name>"}'
```

Response:
```
HTTP/1.1 200 OK
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 15:23:36 GMT
Content-Type: application/json
Content-Length: 0
Connection: keep-alive
Access-Control-Expose-Headers: Location
```

### Delete Thing
To delete a thing you need a `thing_id` and a `user_token`

> Must-have: `user_token` and `thing_id`

```
curl -s -S -i -X DELETE -H "Content-Type: application/json" -H  "Authorization: <user_token>" http://localhost/things/<thing_id>
```

Response:
```
HTTP/1.1 204 No Content
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 15:24:44 GMT
Content-Type: application/json
Connection: keep-alive
Access-Control-Expose-Headers: Location
```

## Channels

### Create Channel
To create a channel, you need a `user_token`

> Must-have: `user_token`

```
curl -s -S -i -X POST -H "Content-Type: application/json" -H "Authorization: <user_token>" http://localhost/channels -d '{"name": "<channel_name>"}'
```

Response:
```
HTTP/1.1 201 Created
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 15:26:51 GMT
Content-Type: application/json
Content-Length: 0
Connection: keep-alive
Location: /channels/db4b7428-e278-4fe3-b85a-d65554d6abe9
Warning-Deprecated: This endpoint will be depreciated in v1.0.0. It will be replaced with the bulk endpoint currently found at /channels/bulk.
Access-Control-Expose-Headers: Location
```

### Create Channels
As with things, you can create multiple channels at once

> Must-have: `user_token` and at least 2 channels

```
curl -s -S -i -X POST -H "Content-Type: application/json" -H "Authorization: <user_token>" http://localhost/channels/bulk -d '[{"name": "<channel_name_1>"}, {"name": "<channel_name_2>"}]'
```

Response:
```
HTTP/1.1 201 Created
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 15:28:10 GMT
Content-Type: application/json
Content-Length: 143
Connection: keep-alive
Access-Control-Expose-Headers: Location

{"channels":[{"id":"b8073d41-01dc-46ad-bb26-cfecc596c6c1","name":"channel_name_1"},{"id":"2200527a-f590-4fe5-b9d6-892fc6f825c3","name":"channel_name_2"}]}
```

### Get Channel
Get a channel entity for a logged in user

> Must-have: `user_token` and `channel_id`

```
curl -s -S -i -X GET -H "Authorization: <user_token>" http://localhost/channels/<channel_id>
```

Response:
```
HTTP/1.1 200 OK
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 15:29:49 GMT
Content-Type: application/json
Content-Length: 63
Connection: keep-alive
Access-Control-Expose-Headers: Location

{"id":"db4b7428-e278-4fe3-b85a-d65554d6abe9","name":"channel_name"}
```

### Get Channels
Get all channels, list requests accepts limit and offset query parameters

> Must-have: `user_token`

```
curl -s -S -i -X GET -H "Authorization: <user_token>" http://localhost/channels
```

Response:
```
HTTP/1.1 200 OK
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 15:30:34 GMT
Content-Type: application/json
Content-Length: 264
Connection: keep-alive
Access-Control-Expose-Headers: Location

{"total":3,"offset":0,"limit":10,"order":"","direction":"","channels":[{"id":"db4b7428-e278-4fe3-b85a-d65554d6abe9","name":"channel_name"},{"id":"b8073d41-01dc-46ad-bb26-cfecc596c6c1","name":"channel_name_1"},{"id":"2200527a-f590-4fe5-b9d6-892fc6f825c3","name":"channel_name_2"}]}
```

### Update Channel
Update channel entity

> Must-have: `user_token` and `channel_id`

```
curl -s -S -i -X PUT -H "Content-Type: application/json" -H  "Authorization: <user_token>" http://localhost/channels/<channel_id> -d '{"name": "<channel_name>"}'
```

Response:
```
HTTP/1.1 200 OK
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 15:32:08 GMT
Content-Type: application/json
Content-Length: 0
Connection: keep-alive
Access-Control-Expose-Headers: Location
```

### Delete Channel
Delete a channel entity

> Must-have: `user_token` and `channel_id`

```
curl -s -S -i -X DELETE -H "Content-Type: application/json" -H  "Authorization: <user_token>" http://localhost/channels/<channel_id>
```

Response:
```
HTTP/1.1 204 No Content
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 15:33:21 GMT
Content-Type: application/json
Connection: keep-alive
Access-Control-Expose-Headers: Location
```

### Connect
Connect thing to channel

> Must-have: `user_token`, `channel_id` and `thing_id`

```
curl -s -S -i -X POST -H "Content-Type: application/json" -H "Authorization: <user_token>" http://localhost/connect -d '{"channel_ids": ["<channel_id>"], "thing_ids": ["<thing_id>"]}'
```

Response:
```
HTTP/1.1 200 OK
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 15:36:32 GMT
Content-Type: application/json
Content-Length: 0
Connection: keep-alive
Access-Control-Expose-Headers: Location
```

### Disconnect
Disconnect the thing from the channel

> Must-have: `user_token`, `channel_id` and `thing_id`

```
curl -s -S -i -X DELETE -H "Content-Type: application/json" -H "Authorization: <user_token>" http://localhost/channels/<channel_id>/things/<thing_id>
```

Response:
```
HTTP/1.1 204 No Content
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 15:38:14 GMT
Content-Type: application/json
Connection: keep-alive
Access-Control-Expose-Headers: Location
```

### Access by Key
Checks if thing has access to a channel

> Must-have: `channel_id` and `thing_key`

```
curl -s -S -i -X POST -H "Content-Type: application/json" http://localhost/identify/channels/<channel_id>/access-by-key -d '{"token": "<thing_key>"}'
```

Response:
```
HTTP/1.1 200 OK
Server: nginx/1.16.0
Date: Mon, 22 Mar 2021 13:10:53 GMT
Content-Type: application/json
Content-Length: 46
Connection: keep-alive
Access-Control-Expose-Headers: Location

{"id":"d69d0098-072b-41bf-8c6e-ce4dbb12d333"}
```

### Access by ID
Checks if thing has access to a channel

> Must-have: `channel_id` and `thing_id`

```
curl -s -S -i -X POST -H "Content-Type: application/json" http://localhost/identify/channels/<channel_id>/access-by-id -d '{"thing_id": "<thing_id>"}'
```

Response:
```
HTTP/1.1 200 OK
Server: nginx/1.16.0
Date: Mon, 22 Mar 2021 15:02:02 GMT
Content-Type: application/json
Content-Length: 0
Connection: keep-alive
Access-Control-Expose-Headers: Location
```

### Identify
Validates thing's key and returns it's ID if key is valid

> Must-have: `thing_key`
```
curl -s -S -i -X POST -H "Content-Type: application/json" http://localhost/identify -d '{"token": "<thing_key>"}'
```

Response:
```
HTTP/1.1 200 OK
Server: nginx/1.16.0
Date: Mon, 22 Mar 2021 15:04:41 GMT
Content-Type: application/json
Content-Length: 46
Connection: keep-alive
Access-Control-Expose-Headers: Location

{"id":"d69d0098-072b-41bf-8c6e-ce4dbb12d333"}
```

## Messages

### Send Messages
Sends message via HTTP protocol

> Must-have: `thing_key` and `channel_id`

```
curl -s -S -i -X POST -H "Content-Type: application/json" -H "Authorization: <thing_key>" http://localhost/http/channels/<channel_id>/messages -d '[{"bn":"some-base-name:","bt":1.276020076001e+09,"bu":"A","bver":5,"n":"voltage","u":"V","v":120.1}, {"n":"current","t":-5,"v":1.2}, {"n":"current","t":-4,"v":1.3}]'
```

Response:
```
HTTP/1.1 202 Accepted
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 16:53:54 GMT
Content-Length: 0
Connection: keep-alive
```

### Read Messages
Reads messages from database for a given channel

> Must-have: `thing_key` and `channel_id`

```
curl -s -S -i -H "Authorization: <thing_key>" http://localhost:<service_port>/channels/<channel_id>/messages?offset=0&limit=5
```

Response:
```
HTTP/1.1 200 OK
Content-Type: application/json
Date: Wed, 10 Mar 2021 16:54:58 GMT
Content-Length: 660

{"offset":0,"limit":10,"format":"messages","total":3,"messages":[{"channel_name":"1a0cde06-8e5c-4f07-aac5-95aff4a19ea0","publisher":"33eb28c3-4ca2-45c3-b1c5-d5d049c6c24e","protocol":"http","name":"some-base-name:voltage","unit":"V","time":1276020076.001,"value":120.1},{"channel_name":"1a0cde06-8e5c-4f07-aac5-95aff4a19ea0","publisher":"33eb28c3-4ca2-45c3-b1c5-d5d049c6c24e","protocol":"http","name":"some-base-name:current","unit":"A","time":1276020072.001,"value":1.3},{"channel_name":"1a0cde06-8e5c-4f07-aac5-95aff4a19ea0","publisher":"33eb28c3-4ca2-45c3-b1c5-d5d049c6c24e","protocol":"http","name":"some-base-name:current","unit":"A","time":1276020071.001,"value":1.2}]}
```

## Groups

### Create group
To create a group, you need the group name and a `user_token` 

> Must-have: `user_token`

```
curl -s -S -i -X POST -H "Content-Type: application/json" -H "Authorization: <user_token>" http://localhost/groups -d '{"name": "<group_name>", "parent_id": "<previous_group_id>", "description": "<group_description>", "metadata": {}}'
```

Response:
```
HTTP/1.1 201 Created
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 16:58:09 GMT
Content-Type: application/json
Content-Length: 0
Connection: keep-alive
Location: /groups/01F0EH61SA7C7NDKWYCXVG7PWD
Access-Control-Expose-Headers: Location
```

### Members
Get list of ID's from group

> Must-have: `user_token` and `group_id`

```
curl -s -S -i -X GET -H 'Content-Type: application/json' -H "Authorization: <user_token>" http://localhost/groups/<group_id>/members  
```

Response:
```
HTTP/1.1 200 OK
Server: nginx/1.16.0
Date: Tue, 23 Mar 2021 09:18:10 GMT
Content-Type: application/json
Content-Length: 116
Connection: keep-alive
Access-Control-Expose-Headers: Location

{"limit":10,"total":0,"level":0,"name":"","Members":[{"ID":"d782b42b-e317-4cd7-9dd0-4e2ea0f349c8","Type":"users"}]}
```

### Assign
Assign user, thing or channel to a group

> Must-have: `user_token`, `group_id`, `member_id` and `member_type`

```
curl -s -S -i -X POST -H "Content-Type: application/json" -H "Authorization: <user_token>" http://localhost/groups/<group_id>/members -d '{"members":["<user_id>" | "<thing_id_>" | "<channel_id_>"], "type":["users" | "things" | "channels"]}' 
```

Response:
```
HTTP/1.1 200 OK
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 17:04:41 GMT
Content-Type: application/json
Content-Length: 0
Connection: keep-alive
Access-Control-Expose-Headers: Location
```

### Unassign
Unassign user, thing or channel from group

> Must-have: `user_token`, `group_id`, `member_id` and `member_type`

```
curl -s -S -i -X DELETE -H "Content-Type: application/json" -H "Authorization: <user_token>" http://localhost/groups/<group_id>/members -d '{"members":["<user_id>" | "<thing_id_>" | "<channel_id_>"], "type":["users" | "things" | "channels"]}'
```

Response:
```
HTTP/1.1 204 No Content
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 17:13:06 GMT
Content-Type: application/json
Connection: keep-alive
Access-Control-Expose-Headers: Location
```

### Get group
Get a group entity for a logged in user

> Must-have: `user_token` and `group_id`

```
curl -s -S -i -X GET -H "Content-Type: application/json" -H "Authorization: <user_token>" http://localhost/groups/<group_id>
```

Response:
```
HTTP/1.1 200 OK
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 17:06:48 GMT
Content-Type: application/json
Content-Length: 201
Connection: keep-alive
Access-Control-Expose-Headers: Location

{"id":"01F0EH61SA7C7NDKWYCXVG7PWD","name":"group_name","owner_id":"d782b42b-e317-4cd7-9dd0-4e2ea0f349c8","level":0,"path":"","created_at":"2021-03-10T16:58:09.579Z","updated_at":"2021-03-10T16:58:09.579Z"}
```

### Get groups
Get all groups, list requests accepts limit and offset query parameters

> Must-have: `user_token`

```
curl -s -S -i -X GET -H "Content-Type: application/json" -H "Authorization: <user_token>" http://localhost/groups
```

Response:
```
HTTP/1.1 200 OK
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 17:09:28 GMT
Content-Type: application/json
Content-Length: 496
Connection: keep-alive
Access-Control-Expose-Headers: Location

{"total":2,"level":0,"name":"","groups":[{"id":"01F0EH61SA7C7NDKWYCXVG7PWD","name":"group_name","owner_id":"d782b42b-e317-4cd7-9dd0-4e2ea0f349c8","level":1,"path":"01F0EH61SA7C7NDKWYCXVG7PWD","created_at":"2021-03-10T16:58:09.579Z","updated_at":"2021-03-10T16:58:09.579Z"},{"id":"01F0EHQTP2HQ7JTWZNMVJ0JJCN","name":"group_name_1","owner_id":"d782b42b-e317-4cd7-9dd0-4e2ea0f349c8","level":1,"path":"01F0EHQTP2HQ7JTWZNMVJ0JJCN","created_at":"2021-03-10T17:07:52.13Z","updated_at":"2021-03-10T17:07:52.13Z"}]}
```

### Update group
Update group entity

> Must-have: `user_token` and `group_id`

```
curl -s -S -i -X PUT -H "Content-Type: application/json" -H  "Authorization: <user_token>" http://localhost/groups/<group_id> -d '{"name": "<group_name>"}'
```

Response:
```
HTTP/1.1 200 OK
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 17:11:51 GMT
Content-Type: application/json
Content-Length: 0
Connection: keep-alive
Access-Control-Expose-Headers: Location
```

### Delete group
Delete a group entity

> Must-have: `user_token` and `group_id`

```
curl -s -S -i -X DELETE -H "Content-Type: application/json" -H "Authorization: <user_token>" http://localhost/groups/<group_id>
```

Response:
```
HTTP/1.1 204 No Content
Server: nginx/1.16.0
Date: Wed, 10 Mar 2021 17:14:13 GMT
Content-Type: application/json
Connection: keep-alive
Access-Control-Expose-Headers: Location
```
