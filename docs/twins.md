## Twins service

*Mainflux twins service is built on top of the Mainflux platform. In order to
fully understand what follows, be sure to get acquainted with [overall Mainflux
architecture](architecture.md).*

**Twin** refers to a **digital representation** of a **real world system**
consisting of possibly multiple data sources/producers and/or
destinations/consumers (data agents).

For example, an industrial machine can use multiple protocols such as MQTT,
OPC-UA, a regularly updated machine hosted CSV file etc. to send measurement
data - such as flowrate, material temperature, etc. - and state metadata - such
as engine and chassis temperature, engine rotations per seconds, identity of the
current human operator, etc. - as well as to receive control, i.e. actuation
messages - such as, turn on/off light, increment/decrement borer speed, switch
knifes, etc.

Each of these data sources and data consumers - which we refer to collectively
here as data agents - can be represented by means of multiple **Mainflux things,
channels and subtopics**. For example, an OPC-UA server can be represented as a
Mainflux thing and different nodes can be represented as multiple Mainflux
channels or multiple subtopics of a single Mainflux channel. A machine hosted
CSV file can be represented as a thing and columns can be represented as
subtopics of a channel.

Although this works well, satisfies the requirements of a wide variety of use
cases and corresponds to the intended use of Mainlfux IoT platform, this setup
can be insufficient in two important ways. Firstly, different things and
channels connections - i.e. Mainflux representations of different data agent
structures - are unrelated to each other, i.e. they do not form a **meaningful
whole** and, as a consequence, they do not represent a **single unified
system**. Secondly, the **semantic** aspect, i.e. the **meaning** of different
things and channels is not transparent and defined by the sole use of Mainflux
platform entities (channels and things).

Certainly, we can try to describe things and channels connections and relations
as well as their meaning - i.e. their role, position, function in the overall
system - by means of their metadata. Although this might work well - with a
proviso of a lot of additional effort of writing the relatively complex code to
create and parse metadata - it is not a practical approach and we still don't
get - at least not out of the box - a readable and useful overview of the system
as a whole and of its components. Also, this approach does not enable us to
answer a simple but very important question, what was the state of the system at
a certain moment in time.

To overcome these problems, Mainflux comes with a **digital twin service**. The
twins service is built on top of the Mainflux platform and relies on its
architecture and entities, more precisely, on Mainflux users, things and
channels. The primary task of the twin service is to handle Mainflux digital
twins. Mainflux digital twin consists of three parts:

- general data about twin itself, i.e. **twin's metadata**,
- history of twin's **definitions**, including current definition,
- history of twin's **states**, including current state.

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

A JSON representation of twin might look like this when we define our digital
twin:

```
{
    "owner": "john.doe@email.net",
    "id": "a838e608-1c1b-4fea-9c34-def877473a89",
    "thing_id": "",
    "name": "grinding machine 2",
    "revision": 2,
    "created": "2020-05-05T08:41:39.142Z",
    "updated": "2020-05-05T08:49:12.638Z",
    "definitions": [
        {
            "id": 0,
            "created": "2020-05-05T08:41:39.142Z",
            "attributes": [],
            "delta": 1000000
        },
        {
            "id": 1,
            "created": "2020-05-05T08:46:23.207Z",
            "attributes": [
                {
                    "name": "engine temperature",
                    "channel": "7ef6c61c-f514-402f-af4b-2401b588bfec",
                    "subtopic": "engine",
                    "persist_state": true
                },
                {
                    "name": "chassis temperature",
                    "channel": "7ef6c61c-f514-402f-af4b-2401b588bfec",
                    "subtopic": "chassis",
                    "persist_state": true
                },
                {
                    "name": "rotations per sec",
                    "channel": "a254032a-8bb6-4973-a2a1-dbf80f181a86",
                    "subtopic": "",
                    "persist_state": false
                }
            ],
            "delta": 1000000
        },
        {
            "id": 2,
            "created": "2020-05-05T08:49:12.638Z",
            "attributes": [
                {
                    "name": "engine temperature",
                    "channel": "7ef6c61c-f514-402f-af4b-2401b588bfec",
                    "subtopic": "engine",
                    "persist_state": true
                },
                {
                    "name": "chassis temperature",
                    "channel": "7ef6c61c-f514-402f-af4b-2401b588bfec",
                    "subtopic": "chassis",
                    "persist_state": true
                },
                {
                    "name": "rotations per sec",
                    "channel": "a254032a-8bb6-4973-a2a1-dbf80f181a86",
                    "subtopic": "",
                    "persist_state": false
                },
                {
                    "name": "precision",
                    "channel": "aed0fbca-0d1d-4b07-834c-c62f31526569",
                    "subtopic": "",
                    "persist_state": true
                }
            ],
            "delta": 1000000
        }
    ]
}
```

In the case of the upper twin, we begin with an empty definition, the one with
the `id` **0** - we could have provided the definition immediately - and over
the course of time, we add two more definitions, so the total number of
revisions is **2**. We decide not to persist the number of rotation per second
in our digital twin state. We define it, though, because the definition and its
attributes are used not only to define states of a complex data agent system,
but also to define the semantic structure of the system. `delta` is the number
of nanoseconds used to determine whether the received attribute value should
trigger the generation of the new state if the value is equal to the previous
value.


Finally, **states** are created according to the twin's current definition. A
state stores twin's ID - every state belongs to a single twin -, state's own ID,
twin's definition sequence number that underlies the state's semantics, creation
date and the actual payload. **Payload** is a set of key-value pairs where a key
corresponds to the attribute name and a value is the actual value of the
attribute. All [SenML value
types](https://tools.ietf.org/html/rfc8428#section-4.3) are supported.

A JSON representation of a partial list of states might look like this:

```
{
    "total": 28,
    "offset": 10,
    "limit": 5,
    "states": [
        {
            "twin_id": "a838e608-1c1b-4fea-9c34-def877473a89",
            "id": 11,
            "definition": 1,
            "created": "2020-05-05T08:49:06.167Z",
            "payload": {
                "chassis temperature": 0.3394171011161684,
                "engine temperature": 0.3814079472715233
            }
        },
        {
            "twin_id": "a838e608-1c1b-4fea-9c34-def877473a89",
            "id": 12,
            "definition": 1,
            "created": "2020-05-05T08:49:12.168Z",
            "payload": {
                "chassis temperature": 1.8116442194724147,
                "engine temperature": 0.3814079472715233
            }
        },
        {
            "twin_id": "a838e608-1c1b-4fea-9c34-def877473a89",
            "id": 13,
            "definition": 2,
            "created": "2020-05-05T08:49:18.174Z",
            "payload": {
                "chassis temperature": 1.8116442194724147,
                "engine temperature": 3.2410616702795814
            }
        },
        {
            "twin_id": "a838e608-1c1b-4fea-9c34-def877473a89",
            "id": 14,
            "definition": 2,
            "created": "2020-05-05T08:49:19.145Z",
            "payload": {
                "chassis temperature": 3.2410616702795814,
                "engine temperature": 3.2410616702795814,
                "precision": 8.922156489392854
            }
        },
        {
            "twin_id": "a838e608-1c1b-4fea-9c34-def877473a89",
            "id": 15,
            "definition": 2,
            "created": "2020-05-05T08:49:24.178Z",
            "payload": {
                "chassis temperature": 0.8694383878692546,
                "engine temperature": 3.2410616702795814,
                "precision": 8.922156489392854
            }
        }
    ]
}
```

As you can see, the first two states correspond to the definition 1 and have
only two attributes in the payload. The rest of the states is based on the
definition 2, where we persist 3 attributes and, as a consequence, its payload
consists of 3 entries.

### Authentication and authorization

Twin belongs to a Mainflux user, tenant that represents physical person or
organization and owns Mainflux things and channels. Mainflux user provides
authorization and authentication mechanisms to twins service. For more details,
please see [Authentication with Mainflux keys](authentication.md). In practical
terms, we need to create a Mainflux user in order to create digital twin. Every
twin belongs to exactly one user. One user can have unlimited number of digital
twins.

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


### Notifications

```
export MF_TWINS_CHANNEL_ID=
```
