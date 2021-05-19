# Rules

## About service

Mainflux Rules service is build around [EMQ X Kuiper](https://github.com/emqx/kuiper), a lightweight IoT rules engine. Mainflux Rules service consists of

1. **Mainflux compatible HTTP API wrapper** for Kuiper, which allows you to perform CRUD operations on **streams** - Kuiper representations of incoming data sources - and **rules** - Kuiper queries used to process (filter and transform) the incoming data.
2. **Mainflux specific adapter** for EMQ X Kuiper which consists of an incoming data adapter (Kuiper's *source*) - for feeding Mainflux platform data into Kuiper - and an outgoing data adapter (Kuiper's *sink*) - for feeding Kuiper processed data back to the Mainflux platform.

### EMQ X Kuiper

On the picture below you can see a Kuiper architecture:

![Kuiper rules engine architecture](img/rules/kuiper_arch.png)

As you can see, the incoming data is handled by **sources** (on the left) and the data is output by **sinks** (on the right).

Mainflux Rules service adds two plugins to Kuiper rules engine, **Mainflux source** plugin and **Mainflux sink** plugin:

![Kuiper rules engine architecture](img/rules/mainflux_kuiper_arch.png)

As you can see from the picture, you use a dedicated Mainflux channel to feed data to Kuiper and a dedicated Mainflux channel to get data from Kuiper. You can use any channel as an input channel. The output channel is unrelated to the input channel so it is up to you to decide whether you will use the same channel or different channels for the source and the sink. 

### About streams, rules, sources and sinks

Kuiper [streams](https://github.com/emqx/kuiper/blob/master/docs/en_US/streams.md) are Kuiper representations of incoming data streams. You use a [subset of an SQL dialect](https://github.com/emqx/kuiper/tree/master/docs/en_US/sqls) to define a stream just as you would define a table in the database. On the other hand, Kuiper [rules](https://github.com/emqx/kuiper/blob/master/docs/en_US/rules/overview.md) are akin to SQL database queries. Again, you use a subset of an SQL dialect to define a query. Query is used to process (filter and transform) the data received *via* stream. 

On the other hand, Kuiper **source** is a Kuiper plugin used to actually receive the data. It is a plugin written in [Go language](https://golang.org/). So, you should not confuse source with a stream. **Source** is a data receiver and **stream** is a data definition (in SQL) - it tells to Kuiper how the data received *via* source looks like. 

Likewise, Kuiper **sink** is a Kuiper plugin used to actually send the data. Like a source plugin, it is also a plugin written in Go language. **Rule** is a query used to process data. If the data, received by a source, satisfies any of the existing queries, it will be passed to a sink. So, you should not confuse sinks, Kuiper data senders, and queries, which are akin to traditional signal filters.

## Start service

You can run Mainflux Rules service either natively or in a docker container. If you decide to run it natively, you will need to provide the working instance of the Kuiper rules engine, and compile the Mainflux [plugins](https://github.com/emqx/kuiper/blob/master/docs/en_US/plugins/overview.md) to run with the instance you have provided.

An easier way to run Mainflux Rules service is to use containers. Mainflux rules service is an addon and can be run as any other Mainflux addon:

```
docker-compose --env-file docker/env -f docker/addons/rules/docker-compose.yml up &
```

This will launch two containers. One container with the Mainflux Rules service and the other with the Kuiper rules engine. Everything is preset so that Rules service communicates with the rules engine. 

## Rules service CRUD operation

For more information about the Rules service HTTP API please refer to the [rules service OpenAPI file](https://github.com/mainflux/mainflux/tree/master/rules/openapi.yml).

### Preparation

See [provision](provision.md)

To quickly provision a system with a user, two channels and two things, do

```
mainflux-cli provision test
```

Response

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

You can use the upper values to populate the environment variables in order to use the Rules service API in a more intuitive way.

```
TOKEN=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2MjA0MjgxODAsImlhdCI6MTYyMDM5MjE4MCwiaXNzIjoibWFpbmZsdXguYXV0aCIsInN1YiI6InBlbnNpdmVfZ2FsaWxlb0BlbWFpbC5jb20iLCJpc3N1ZXJfaWQiOiI0NDlhMGE5YS04YjE5LTQwMTgtODA3My05YzZjMmM2ZDhjYzkiLCJ0eXBlIjowfQ.QFjP27WAAy8OUt6yQzjLjk8HVO9CMeKNQThwYYHcPIA \
THING_ID=4544b5b7-09c5-4f7c-b3c4-47611efd7bed \
THING_KEY=9a794000-968b-4abb-a881-ce41ec606f67 \
THING_ID2=54b4716e-3c59-4757-8dfc-bbd3b389ad09 \
THING_KEY2=0819d46c-36c2-4996-9339-b873613dc103 \
CHANNEL_ID=f9e0c84a-0077-4dbd-819b-3df78b1e1c3d \
CHANNEL_ID2=e1af816a-dfdd-47d9-8ab2-25e8a2f7e8a6
```

The rest of the document will assume that the upper environment variables are defined. 

Once you have defined the environment variables, make sure to connect a second thing with a second channel. Namely, `mainflux-cli provision test` connects the first thing with both channels and the second thing with the first channel.

```
mainflux-cli things connect $THING_ID2 $CHANNEL_ID2 $TOKEN
```

### Kuiper status

To query the Kuiper rules engine status

```
curl -i -X GET http://localhost:9099/info
```

Response

```
{"version":"1.1.2-24-gd1122ee","os":"linux","upTimeSeconds":82}
```

### Streams

#### Create

To create a Kuiper stream

```
curl -i -X POST -H "Content-Type: application/json" -H "Authorization: $TOKEN" localhost:9099/streams -d '{"name":"demo", "row":"v float, n string", "host":"nats://nats", "port":"4222", "channel": "'$CHANNEL_ID'", "subtopic": "motor"}'
```

The field `name` uniquely identifies the stream. The field `row` describes the names and the data types of the messages that compose the incoming stream of messages.

Response

```
{"result":"Create stream successful."}
```

#### List

To get a list of owned streams

```
curl -i -X GET -H "Authorization: $TOKEN" http://localhost:9099/streams
```

Response

```
{"streams":["demo"]}
```

#### View

To view a particular stream

```
curl -i -X GET -H "Authorization: $TOKEN" http://localhost:9099/streams/<stream_name>
```

E.g.

```
curl -i -X GET -H "Authorization: $TOKEN" http://localhost:9099/streams/demo
```

Response

```
{"Stream":{"Name":"demo","StreamFields":[{"Name":"v","FieldType":"float"},{"Name":"n","FieldType":"string"}],"Options":{"DATASOURCE":"nats://nats:4222;2ca5aa2d-305d-4517-8311-57726c043172.motor","FORMAT":"json"}}}
```

Due to a Kuiper API, the field `DATASOURCE` is a compound out of Kuiper rules engine address, `nats://nats:4222`, Mainflux channel id, `2ca5aa2d-305d-4517-8311-57726c043172`, and a subtopic, `motor`.

The field `StreamFields` contains the SQL description of the incoming stream of messages.

#### Update

To update a particular stream, use the same format used to create a stream, with new values. The stream is identified with a `<stream_name>` value.

```
curl -i -X PUT -H "Content-Type: application/json" -H "Authorization: $TOKEN" localhost:9099/streams/<stream_name> -d '{"name":"<stream_name>", "row":"v1 float, n1 string", "host":"nats://localhost", "port":"4222", "channel": "'$CHANNEL_ID'", "subtopic": "motor"}'
```

E.g.

```
curl -i -X PUT -H "Content-Type: application/json" -H "Authorization: $TOKEN" localhost:9099/streams/demo -d '{"name":"demo", "row":"v1 float, n1 string", "host":"nats://localhost", "port":"4222", "channel": "'$CHANNEL_ID'", "subtopic": "motor"}'
```

Response

```
{"result":"Update stream successful."}
```

#### Delete

To delete a stream

```
curl -i -X DELETE -H "Authorization: $TOKEN" http://localhost:9099/streams/<stream_name>
```

E.g.

```
curl -i -X DELETE -H "Authorization: $TOKEN" http://localhost:9099/streams/demo
```

Response

```
{"result":"Delete streams successful."}
```

### Rules

#### Create

To create a Kuiper rule

```
curl -i -X POST -H "Content-Type: application/json" -H "Authorization: $TOKEN" localhost:9099/rules -d '{ "id": "demo", "sql": "select * from demo where v > 1.2;", "host": "nats://nats", "port": "4222", "channel": "'$CHANNEL_ID2'", "subtopic": "engine", "send_meta_to_sink": true }'
```

The field `id` uniquely identifies the rule. The field `sql` defines the SQL query rule: it is used to query the incoming stream of messages for messages satisfying a certain condition. In the example, we use `select * from demo where v > 1.2`. `demo` here refers to the name of the stream and it is not to be confused with the `id` value, a name of the rule. The names of streams and rules are arbitrary, do not need to match, but must be unique across the streams and rules respectively.

Response

```
{"result":"Create rule successful."}
```

#### List

To get a list of owned rules

```
curl -i -X GET -H "Authorization: $TOKEN" http://localhost:9099/rules
```

Response

```
{"Rules":[{"id":"demo","status":"Running"}]}
```

Pay attention to the status of the rule. It shoud be `Running`. Otherwise, there is some problem with the setup.

#### View

To view a particular rule

```
curl -i -X GET -H "Authorization: $TOKEN" http://localhost:9099/rules/<rule_id>
```

For example

```
curl -i -X GET -H "Authorization: $TOKEN" http://localhost:9099/rules/demo
```

Response

```
{"Rule":{"id":"demo","sql":"select * from demo where v \u003e 1.2;","Actions":[{"mainflux":{"host":"nats://nats","port":"4222","channel":"0f08de5a-b7e4-4c52-b4c7-837b0bf84e08","subtopic":"engine"}}],"Options":{"SendMetaToSink":true}}}
```

The field `Actions` contains the Kuiper rules engine internal information related to the rule. The subfield `Options` contains the object with the the rules options. TODO: `SendMetaToSink`.

#### Update

To update a particular rule, use the same format used to create a rule, but with new values. The rule is identified with a `<rule_id>` value.

```
curl -i -X PUT -H "Content-Type: application/json" -H "Authorization: $TOKEN" http://localhost:9099/rules/<rule_id> -d '{ "id": "<rule_id>", "sql": "select * from demo where v > 1.25;", "host": "nats://nats", "port": "4222", "channel": "'$CHANNEL_ID2'", "subtopic": "engine", "send_meta_to_sink": false }}'
```
E.g.

```
curl -i -X PUT -H "Content-Type: application/json" -H "Authorization: $TOKEN" http://localhost:9099/rules/demo -d '{ "id": "demo", "sql": "select * from demo where v > 1.25;", "host": "nats://nats", "port": "4222", "channel": "'$CHANNEL_ID2'", "subtopic": "engine", "send_meta_to_sink": false }}'
```

Response

```
{"result":"Update rule successful."}
```

#### Delete

To delete a rule

```
curl -i -X DELETE -H "Authorization: $TOKEN" http://localhost:9099/rules/<rule_id>
```

E.g.

```
curl -i -X DELETE -H "Authorization: $TOKEN" http://localhost:9099/rules/demo
```

#### Status

To get the rule's status, i.e. the metrics of the running rule

```
curl -i -X GET -H "Authorization: $TOKEN" http://localhost:9099/rules/demo/<rule_id>
```

E.g.

```
curl -i -X GET -H "Authorization: $TOKEN" http://localhost:9099/rules/demo/status
```

Response

```
{"op_1_preprocessor_demo_0_buffer_length":0,"op_1_preprocessor_demo_0_exceptions_total":0,"op_1_preprocessor_demo_0_last_invocation":0,"op_1_preprocessor_demo_0_process_latency_us":0,"op_1_preprocessor_demo_0_records_in_total":0,"op_1_preprocessor_demo_0_records_out_total":0,"op_2_filter_0_buffer_length":0,"op_2_filter_0_exceptions_total":0,"op_2_filter_0_last_invocation":0,"op_2_filter_0_process_latency_us":0,"op_2_filter_0_records_in_total":0,"op_2_filter_0_records_out_total":0,"op_3_project_0_buffer_length":0,"op_3_project_0_exceptions_total":0,"op_3_project_0_last_invocation":0,"op_3_project_0_process_latency_us":0,"op_3_project_0_records_in_total":0,"op_3_project_0_records_out_total":0,"sink_mainflux_0_0_buffer_length":0,"sink_mainflux_0_0_exceptions_total":0,"sink_mainflux_0_0_last_invocation":0,"sink_mainflux_0_0_process_latency_us":0,"sink_mainflux_0_0_records_in_total":0,"sink_mainflux_0_0_records_out_total":0,"source_demo_0_buffer_length":0,"source_demo_0_exceptions_total":0,"source_demo_0_last_invocation":0,"source_demo_0_process_latency_us":0,"source_demo_0_records_in_total":0,"source_demo_0_records_out_total":0}
```

#### Control

In order to start, stop and restart the rule, you always use the command of the form

```
curl -i -X POST -H "Authorization: $TOKEN" POST http://localhost:9099/rules/<rule_id>/<action>
```

Where `action` can be on `start`, `stop` and `restart`.

Start the rule

```
curl -i -X POST -H "Authorization: $TOKEN" POST http://localhost:9099/rules/demo/start
```

Response

```
{"result":"start successful."}
```

Stop the rule

```
curl -i -X POST -H "Authorization: $TOKEN" POST http://localhost:9099/rules/demo/stop
```

Response

```
{"result":"stop successful."}
```

Restart the rule

```
curl -i -X POST -H "Authorization: $TOKEN" POST http://localhost:9099/rules/demo/restart
```

Response

```
{"result":"restart successful."}
```

## Demo

[Prepare](#preparation) the environment variables.

`cd` to `docker` and launch Rules service with

```
docker-compose --env-file .env -f addons/rules/docker-compose.yml up &
```

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
