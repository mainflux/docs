# API

## Create User
To start working with the Mainflux system, you need to create a user account.

> Must-have: e-mail and password (password must contain at least 8 chars)

```
curl -s -S -i -X POST -H "Content-Type: application/json" http://localhost/users -d '{"email":"test@email.com", "password":"12345678"}'
```

## Create token
To log in to the Mainflux system, you need to create a token.

> Must-have: registered e-mail and password

```
curl -s -S -i -X POST -H "Content-Type: application/json" http://localhost/tokens -d '{"email":"test@email.com", "password":"12345678"}'
```

## Get user
You can always check the user entity that is logged in by entering the user ID and token.

> Must-have: userID and token

```
curl -s -S -i -X GET -H "Authorization: token" http://localhost/users/userID
```

## Get all users
You can get all users in the database by calling the this function

> Must-have: token

```
curl -s -S -i -X GET -H "Authorization: token" http://localhost/users
```

## Update user
Updating user entity

> Must-have: token, e-mail and password
> Nice-to-have: metadata

```
curl -s -S -i -X PUT -H "Content-Type: application/json" -H "Authorization: token" http://localhost/users -d '{"email":"test@email.com", "password":"12345678"}'
```
## Change password
Changing the user password can be done by calling the update password function

> Must-have: token, old_password and password(new password)

```
curl -s -S -i -X PATCH -H "Content-Type: application/json" -H "Authorization: token" http://localhost/password -d '{"old_password":"12345678", "password":"87654321"}'
```

## Create thing
To create a thing, you need the thing and a token

>Must-have: token and thing
>Should-have: thing key
>Nice-to-have: metadata

```
curl -s -S -i -X POST -H "Content-Type: application/json" -H "Authorization: token" http://localhost/things/bulk -d '[{"name": "thing", "key": ""}]'
```

## Create things
You can create multiple things at once by entering a series of things structures and a token
> Must-have: token and at least 2 thing with names
> Should-have: thing key
> Nice-to-have: metadata

```
curl -s -S -i -X POST -H "Content-Type: application/json" -H "Authorization: token" http://localhost/things/bulk -d '[{"name": "", "key": ""}, {"name": "", "key": ""}]'
```

## Get thing
You can get thing entity by entering the thing ID and token

> Must-have: token and thingID

```
curl -s -S -i -X GET -H "Authorization: token" http://localhost/things/thingID
```

## Get all things
Get all things, list requests accepts limit and offset query parameters

> Must-have: token
> Should-have: query parameters

```
curl -s -S -i -X GET -H "Authorization: token" http://localhost/things
```

## Update thing
Updating a thing entity

> Must-have: token and thingID
> Nice-to-have: thing and metadata

```
curl -s -S -i -X PUT -H "Content-Type: application/json" -H  "Authorization: token" http://localhost/things/thingID -d '{"name": "thing"}'
```

## Delete thing
To delete a thing you need a thing ID and a token

> Must-have: token and thingID

```
curl -s -S -i -X DELETE -H "Content-Type: application/json" -H  "Authorization: token" http://localhost/things/ThingID
```

## Connect
Connect the thing to the channel

> Must-have: token, channelID and thingID

```
curl -s -S -i -X POST -H "Content-Type: application/json" -H "Authorization: token" http://localhost/connect -d '{"channel_ids": ["chanelID"], "thing_ids": ["thingID"]}'
```

## Disconnect
Disconnect the thing from the channel

> Must-have: token, channelID and thingID

```
curl -s -S -i -X DELETE -H "Content-Type: application/json" -H "Authorization: token" http://localhost/channels/channelID/things/thingID
```

## Create channel
To create a channel, you need a channel and a token

> Must-have: token and channel
> Nice-to-have: metadata

```
curl -s -S -i -X POST -H "Content-Type: application/json" -H "Authorization: token" http://localhost/channels -d '{"name": "channel"}'
```

## Create channels
As with things, you can create multiple channels at once

> Must-have: token and at least 2 channels
> Nice-to-have: metadata

```
curl -s -S -i -X POST -H "Content-Type: application/json" -H "Authorization: token" http://localhost/channels/bulk -d '[{"name": "", "key": ""}, {"name": "", "key": ""}]'
```

## Get channel
Get a channel entity for a logged in user

> Must-have: token and channelID
```
curl -s -S -i -X GET -H "Authorization: token" http://localhost/channels/channelID
```

## Get channels
Get all channels, list requests accepts limit and offset query parameters

> Must-have: token
> Should-have: query parameters

```
curl -s -S -i -X GET -H "Authorization: token" http://localhost/channels
```

## Update channel
Update channel entity

> Must-have: token and channelID
> Nice-to-have: channel and metadata

```
curl -s -S -i -X PUT -H "Content-Type: application/json" -H  "Authorization: token" http://localhost/channels/channelID -d '{"name": "channel"}'
```

## Delete channel
Delete a channel entity

> Must-have: token and channelID

```
curl -s -S -i -X DELETE -H "Content-Type: application/json" -H  "Authorization: token" http://localhost/channels/channelID
```

## Send messages
Sends message via HTTP protocol

> Must-have: token and channelID
> Should-have: message

```
curl -s -S -i -X POST -H "Content-Type: application/json" -H "Authorization: tokenn (thing key)" http://localhost/http/channels/channelID/messages -d '[{"bn":"some-base-name:","bt":1.276020076001e+09, "bu":"A","bver":5, "n":"voltage","u":"V","v":120.1}, {"n":"current","t":-5,"v":1.2}, {"n":"current","t":-4,"v":1.3}]'
```

## Read messages
Reads messages from database for a given channel

> Must-have: token and channelID

```
curl -s -S -i -X POST -H "Content-Type: application/json" -H "Authorization: tokenn (thing key)" http://localhost/http/channels/channelID/messages -d '[{"bn":"some-base-name:","bt":1.276020076001e+09, "bu":"A","bver":5, "n":"voltage","u":"V","v":120.1}, {"n":"current","t":-5,"v":1.2}, {"n":"current","t":-4,"v":1.3}]'
```

## Create group
To create a group, you need the group name and a token 

> Must-have: token and group name
> Nice-to-have: parent_id, description, metadata

```
curl -s -S -i -X POST -H "Content-Type: application/json" -H "Authorization: token" http://localhost/groups -d '{"name": "group", "parent_id": "", "descrition": "", "metadata": {}}'
```

## Delete group
Delete a group entity

> Must-have: token and groupID

```
curl -s -S -i -X DELETE -H "Content-Type: application/json" -H "Authorization: token" http://localhost/groups/groupID
```
## Members

curl -s -S -i -X http://localhost/groups/groupID/members -H 'Content-Type: application/json' -H "Authorization: token" 

> Must-have: token and groupID

## Assign
Assign user to group

```
curl -s -S -i -X POST -H "Content-Type: application/json" -H "Authorization: token" http://localhost/groups/groupID/members -d '{"members":["userID"],"type":"users"}' 
```

## Unassign
Unassign user from group

```
curl -s -S -i -X DELETE -H "Content-Type: application/json" -H "Authorization: token" http://localhost/groups/groupID/members -d '{"members":["userID"],"type":"users"}'
```

## Get group
Get a group entity for a logged in user

> Must-have: token and groupID

```
curl -s -S -i -X GET -H "Authorization: token" http://localhost/groups/groupID
```

## Get groups
Get all groups, list requests accepts limit and offset query parameters

> Must-have: token
> Should-have: query parameters

```
curl -s -S -i -X GET -H "Authorization: token" http://localhost/groups
```

## Update group
Update group entity

> Must-have: token and groupID
> Nice-to-have: description and metadata

```
curl -s -S -i -X PUT -H "Content-Type: application/json" -H  "Authorization: token" http://localhost/groups/groupID -d '{"name": "group"}'
```