## Twins service

*Mainflux twins service is built on top of the Mainflux platform. In order to
fully understand what follows, be sure to get acquainted with [overall Mainflux
architecture](architecture.md).*

**Twin** refers to a **digital representation** of the **real world system**
consisting of possibly multiple data sources/producers and/or
destinations/consumers (data agents).

For example, an industrial machine can use multiple protocols such as MQTT,
OPC-UA, a regularly updated machine hosted CSV file etc. to send measurement
data and state metadata as well as to receive control (actuating) messages. 

Each of these data sources and data consumers (data agents) can be represented
by means of multiple **Mainflux things, channels and subtopics**. For example,
an OPC-UA server can be represented as a Mainflux thing and different nodes can
be represented as multiple Mainflux channels or subtopics of a single channel. A
machine hosted CSV file can be represented as a thing and columns can be
represented as subtopics of a channel.

Although this works well, this setup is limited in two important ways. Firstly,
different things and channels connections (setups) are unrelated to each other,
i.e. they do not form a **meaningful whole** and, as a consequence, they do not
represent a **single unified system**. Secondly, the **semantic** aspect, i.e.
the **meaning** of different things and channels is not defined by default.

Certainly, we can try to inscribe things and channels as well as their meaning -
role, position, function in the overall system - into their **metadata**.
Although this might work - with a lot of additional effort of writing the
relatively complex code to parse things' and channels' metadata - it is not a
practical approach and we still don't get - at least not out of the box - a nice
overview of the system as a whole and of its components.

To overcome these two problems, Mainflux comes with a **digital twin service**.
Mainflux digital twin consists of three parts:

- general data about twin itself, i.e. **twin's metadata**,
- history of twin's **definitions**, including current definition,
- history of twin's **states**, including current state.

### Authentication and authorization

Twin belongs to a Mainflux user, tenant that represents physical person or
organization and owns Mainflux things and channels. Mainflux user provides
authorization and authentication mechanisms to twins service. For more details,
please see [Authentication with Mainflux keys](authentication.md).

## Twin's anatomy

Twin's **general information** stores twin's owner email - owner is represented
by Mainflux user -, twin's ID (unique) and name (not unique), twin's creation
and update dates as well as twin's revision number. The latter refers to the
sequential number of twin's definition.

The twin's **definition** is meant to be a semantic representation of system's
data sources and consumers (data agents). Each data data agent is represented by
means of **attribute**. Attribute consists of data agent's name, Mainflux
channel and subtopic over which it communicates. Nota bene: each attribute is
uniquely defined by the combination of channel and subtopic and we cannot have
two or more attributes as a part of the same definition with a same channel and
subtopic.

Attributes has a state persistance flag that determines whether the messages
communicated by it's corresponding channel and subtopic trigger the creation of
a new twin state.

A JSON representation of twin might look like this:

```
{
    "owner": "john.doe@email.net",
    "id": "e7ab8602-4069-445d-9897-b4ef30887fa6",
    "name": "grinding machine 2",
    "revision": 2,
    "created": "2020-05-04T11:15:42.839Z",
    "updated": "2020-05-04T11:18:55.674Z",
    "definitions": [
        {
            "id": 0,
            "created": "2020-05-04T11:15:42.839Z",
            "attributes": [
                {
                    "name": "engine temperature",
                    "channel": "3b57b952-318e-47b5-b0d7-a14f61ecd03b",
                    "subtopic": "engine",
                    "persist_state": true
                },
                {
                    "name": "chassis temperature",
                    "channel": "3b57b952-318e-47b5-b0d7-a14f61ecd03b",
                    "subtopic": "chassis",
                    "persist_state": true
                },
                {
                    "name": "rotations per sec",
                    "channel": "94d6ed5b-cbd7-4cdd-bac8-8dec385e43ec",
                    "subtopic": "",
                    "persist_state": true
                },
                {
                    "name": "precision",
                    "channel": "94d6ed5b-cbd7-4cdd-bac8-8dec385e43ec",
                    "subtopic": "",
                    "persist_state": false
                }
            ],
            "delta": 3
        },
        {
            "id": 1,
            "created": "2020-05-12T11:14:42.543Z",
            "attributes": [
                {
                    "name": "engine temperature",
                    "channel": "3b57b952-318e-47b5-b0d7-a14f61ecd03b",
                    "subtopic": "engine",
                    "persist_state": true
                },
                {
                    "name": "chassis temperature",
                    "channel": "3b57b952-318e-47b5-b0d7-a14f61ecd03b",
                    "subtopic": "chassis",
                    "persist_state": true
                },
                {
                    "name": "rotations per sec",
                    "channel": "94d6ed5b-cbd7-4cdd-bac8-8dec385e43ec",
                    "subtopic": "",
                    "persist_state": false
                },
            ],
            "delta": 10
        },
    ]
}
```

Finally, **states** are created according to the twin's current definition. A
state stores twin's ID - every state belongs to a single twin -, state's own ID,
twin's definition sequence number that underlies the state's semantics, creation
date and the actual payload. **Payload** is a set of key-value pairs where a key
corresponds to the attribute name and a value is the actual value of the
attribute. All [SenML value
types](https://tools.ietf.org/html/rfc8428#section-4.3) are supported.


### CRUD operations



#### Create && Update

Create and update request

```
{
  "name": "twin_name",
  "definition": {
    "attributes": [
      {
        "name": "temperature",
        "channel": "3b57b952-318e-47b5-b0d7-a14f61ecd03b",
        "subtopic": "temperature",
        "persist_state": true
      },
      {
        "name": "humidity",
        "channel": "3b57b952-318e-47b5-b0d7-a14f61ecd03b",
        "subtopic": "humidity",
        "persist_state": false
      },
      {
        "name": "pressure",
        "channel": "3b57b952-318e-47b5-b0d7-a14f61ecd03b",
        "subtopic": "pressure",
        "persist_state": true
      }
    ],
    "delta": 1
  }
}
```


```
curl -s -S -i -X POST -H "Content-Type: application/json" -H "Authorization: <user_auth_token>" http://localhost:8191/twins -d '<twin_data>'
```


```
curl -s -S -i -X PUT -H "Content-Type: application/json" -H "Authorization: <user_auth_token>" http://localhost:8191/<twin_id> -d '<twin_data>'
```

#### View

```
curl -s -S -i -X GET -H "Authorization: <user_auth_token>" http://localhost:8191/twins/<twin_id>
```

#### List

```
curl -s -S -i -X GET -H "Authorization: <user_auth_token>" http://localhost:8191/twins
```


#### Delete

```
curl -s -S -i -X DELETE -H "Authorization: <user_auth_token>" http://localhost:8191/twins/<twin_id>
```

### STATES operations

#### List

```
curl -s -S -i -X GET -H "Authorization: <user_auth_token>" http://localhost:8191/states/<twin_id>
```
