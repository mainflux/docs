## Components

Mainflux IoT platform is comprised of the following services:

| Service                                                                   | Description                                                                      |
|:--------------------------------------------------------------------------|:---------------------------------------------------------------------------------|
| [users](https://github.com/mainflux/mainflux/tree/master/users)           | Manages platform's users and auth concerns                                       |
| [things](https://github.com/mainflux/mainflux/tree/master/things)         | Manages platform's things, channels and access policies                          |
| [http-adapter](https://github.com/mainflux/mainflux/tree/master/http)     | Provides an HTTP interface for accessing communication channels                  |
| [mqtt-adapter](https://github.com/mainflux/mainflux/tree/master/mqtt)     | Provides an MQTT and MQTT over WS interface for accessing communication channels |
| [coap-adapter](https://github.com/mainflux/mainflux/tree/master/coap)     | Provides a CoAP interface for accessing communication channels                   |
| [opcua-adapter](https://github.com/mainflux/mainflux/tree/master/opcua)   | Provides an OPC-UA interface for accessing communication channels                |
| [lora-adapter](https://github.com/mainflux/mainflux/tree/master/lora)     | Provides a LoRa Server forwarder for accessing communication channels            |
| [mainflux-cli](https://github.com/mainflux/mainflux/tree/master/cli)      | Command line interface                                                           |

![arch](img/architecture.jpg)

## Domain Model

The platform is built around 3 main entities: **users**, **things** and **channels**.

`User` represents the real (human) user of the system. It is represented via its
e-mail and password, which he uses as platform access credentials in order to obtain
an access token. Once logged into the system, user can manage his resources (i.e.
things and channels) in CRUD fashion and define access control policies by
connecting them.

`Thing` represents devices (or applications) connected to Mainflux that uses the
platform for message exchange with other "things".

`Channel` represents a communication channel. It serves as message topic that
can be consumed by all of the things connected to it.

## Messaging

Mainflux uses [NATS](https://nats.io) as its messaging backbone, due to its
lightweight and performant nature. You can treat its *subjects* as physical
representation of Mainflux channels, where subject name is constructed using
channel unique identifier.

In general, there is no constrained put on content that is being exchanged
through channels. However, in order to be post-processed and normalized,
messages should be formatted using [SenML](https://tools.ietf.org/html/draft-ietf-core-senml-08).

## Edge

Mainflux platform can be run on the edge as well. Deploying Mainflux on a gateway makes it able to collect, store and analyze data, organize and authenticate devices.
To connect Mainflux instances running on a gateway with Mainflux in a cloud we can use two gateway services developed for that purpose:

* [Agent](agent.md)
* [Export](export.md)

## Unified IoT Platform
Running Mainflux on gateway moves computation from cloud towards the edge thus decentralizing IoT system. 
Since we can deploy same Mainflux code on gateway and in the cloud there are many benefits but the biggest one is easy deployment and adoption - once the engineers understand how to deploy and maintain the platform, they will have the same known work across the whole edge-fog-cloud continuum.
Same set of tools can be used, same patches and bug fixes can be applied. The whole system is much easier to reason about, and the maintenance is much easier and less costly.
