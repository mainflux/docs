# Rules

## About service

## Start service



## Rules service CRUD operation

For more information about the Rules service HTTP API please refer to the [rules service OpenAPI file](https://github.com/mainflux/mainflux/tree/master/rules/openapi.yml).

### Preparation

See [provision](provision.md)

```
mainflux-cli provision test
```

```
{
  "email": "pensive_galileo@email.com",
  "password": "12345678"
}


"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2MjA0MjgxODAsImlhdCI6MTYyMDM5MjE4MCwiaXNzIjoibWFpbmZsdXguYXV0aCIsInN1YiI6InBlbnNpdmVfZ2FsaWxlb0BlbWFpbC5jb20iLCJpc3N1ZXJfaWQiOiI0NDlhMGE5YS04YjE5LTQwMTgtODA3My05YzZjMmM2ZDhjYzkiLCJ0eXBlIjowfQ.QFjP27WAAy8OUt6yQzjLjk8HVO9CMeKNQThwYYHcPIA"


[
  {
    "id": "4544b5b7-09c5-4f7c-b3c4-47611efd7bed",
    "key": "9a794000-968b-4abb-a881-ce41ec606f67",
    "name": "d0"
  },
  {
    "id": "54b4716e-3c59-4757-8dfc-bbd3b389ad09",
    "key": "0819d46c-36c2-4996-9339-b873613dc103",
    "name": "d1"
  }
]


[
  {
    "id": "f9e0c84a-0077-4dbd-819b-3df78b1e1c3d",
    "name": "c0"
  },
  {
    "id": "e1af816a-dfdd-47d9-8ab2-25e8a2f7e8a6",
    "name": "c1"
  }
]
```

```
TOKEN=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2MjA0MjgxODAsImlhdCI6MTYyMDM5MjE4MCwiaXNzIjoibWFpbmZsdXguYXV0aCIsInN1YiI6InBlbnNpdmVfZ2FsaWxlb0BlbWFpbC5jb20iLCJpc3N1ZXJfaWQiOiI0NDlhMGE5YS04YjE5LTQwMTgtODA3My05YzZjMmM2ZDhjYzkiLCJ0eXBlIjowfQ.QFjP27WAAy8OUt6yQzjLjk8HVO9CMeKNQThwYYHcPIA \
THING_ID=4544b5b7-09c5-4f7c-b3c4-47611efd7bed \
THING_KEY=9a794000-968b-4abb-a881-ce41ec606f67 \
THING_ID2=54b4716e-3c59-4757-8dfc-bbd3b389ad09 \
THING_KEY2=0819d46c-36c2-4996-9339-b873613dc103 \
CHANNEL_ID=f9e0c84a-0077-4dbd-819b-3df78b1e1c3d \
CHANNEL_ID2=e1af816a-dfdd-47d9-8ab2-25e8a2f7e8a6
```

```
mainflux-cli things connect $THING_ID2 $CHANNEL_ID2 $TOKEN
```

### Kuiper status

```
curl -i -X GET http://localhost:9099/info
```

### Streams
#### Create

```
curl -i -X POST -H "Content-Type: application/json" -H "Authorization: $TOKEN" localhost:9099/streams -d '{"name":"demo", "row":"v float, n string", "host":"nats://nats", "port":"4222", "channel": "'$CHANNEL_ID'", "subtopic": "motor"}'
```

#### List

```
curl -i -X GET -H "Authorization: $TOKEN" http://localhost:9099/streams
```

#### View

```
curl -i -X GET -H "Authorization: $TOKEN" http://localhost:9099/streams/demo
```

#### Update

```
curl -i -X PUT -H "Content-Type: application/json" -H "Authorization: $TOKEN" localhost:9099/streams/demo -d '{"name":"demo", "row":"v1 float, n1 string", "host":"nats://localhost", "port":"4222", "channel": "'$CHANNEL_ID'", "subtopic": "motor"}'
```

#### Delete

```
curl -i -X DELETE -H "Authorization: $TOKEN" http://localhost:9099/streams/demo
```



### Rules
#### Create

```
curl -i -X POST -H "Content-Type: application/json" -H "Authorization: $TOKEN" localhost:9099/rules -d '{ "id": "demo", "sql": "select * from demo where v > 1.2;", "host": "nats://nats", "port": "4222", "channel": "'$CHANNEL_ID2'", "subtopic": "engine", "send_meta_to_sink": true }'
```

#### List

```
curl -i -X GET -H "Authorization: $TOKEN" http://localhost:9099/rules
```

#### View

```
curl -i -X GET -H "Authorization: $TOKEN" http://localhost:9099/rules/demo
```

#### Update

```
curl -i -X PUT -H "Content-Type: application/json" -H "Authorization: $TOKEN" http://localhost:9099/rules/demo -d '{ "id": "demo", "sql": "select * from demo where v > 1.25;", "host": "nats://nats", "port": "4222", "channel": "'$CHANNEL_ID2'", "subtopic": "engine", "send_meta_to_sink": false }}'
```

#### Delete

```
curl -i -X DELETE -H "Authorization: $TOKEN" http://localhost:9099/rules/demo
```

#### Status

```
curl -i -X GET -H "Authorization: $TOKEN" http://localhost:9099/rules/demo/status
```

#### Control

Start the rule

```
curl -i -X POST -H "Authorization: $TOKEN" POST http://localhost:9099/rules/demo/start
```

Stop the rule

```
curl -i -X POST -H "Authorization: $TOKEN" POST http://localhost:9099/rules/demo/stop
```

Restart the rule

```
curl -i -X POST -H "Authorization: $TOKEN" POST http://localhost:9099/rules/demo/restart
```


## Demo

[Prepare](#preparation) the environment variables.

Create a stream

```
curl -i -X POST -H "Content-Type: application/json" -H "Authorization: $TOKEN" localhost:9099/streams -d '{"name":"demo", "row":"v float, n string", "host":"nats://nats", "port":"4222", "channel": "'$CHANNEL_ID'", "subtopic": "motor"}'
```

Create a rule

```
curl -i -X POST -H "Content-Type: application/json" -H "Authorization: $TOKEN" localhost:9099/rules -d '{ "id": "demo", "sql": "select * from demo where v > 1.2;", "host": "nats://nats", "port": "4222", "channel": "'$CHANNEL_ID2'", "subtopic": "engine", "send_meta_to_sink": true }'
```

Set messaging related environment variables

```
URL=localhost \
MESSAGE='[{"bn":"some-base-name:", "bu":"A","bver":5, "n":"voltage","u":"V","v":1.1}, {"n":"current","v":1.22}, {"n":"current","v":1.3}]'
```

Subscribe to an mqtt broker

```
mosquitto_sub -u ${THING_ID2} -P ${THING_KEY2} -t channels/${CHANNEL_ID2}/messages/engine -h ${URL}
```

Send a message via mqtt

```
mosquitto_pub -u ${THING_ID} -P ${THING_KEY} -t channels/${CHANNEL_ID}/messages/motor -h ${URL} -m "${MESSAGE}"
```

```
[{"bn":"rules_engine","n":"some-base-name:current","v":1.22}]
[{"bn":"rules_engine","n":"some-base-name:current","v":1.3}]
```

Update the rule

```
curl -i -X PUT -H "Content-Type: application/json" -H "Authorization: $TOKEN" http://localhost:9099/rules/demo -d '{ "id": "demo", "sql": "select * from demo where v > 1.25;", "host": "nats://nats", "port": "4222", "channel": "'$CHANNEL_ID2'", "subtopic": "engine", "send_meta_to_sink": false }}'
```

Send a message via mqtt

```
mosquitto_pub -u ${THING_ID} -P ${THING_KEY} -t channels/${CHANNEL_ID}/messages/motor -h ${URL} -m "${MESSAGE}"
```

```
[{"bn":"rules_engine","n":"some-base-name:current","v":1.3}]
```
