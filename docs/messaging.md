Once a channel is provisioned and thing is connected to it, it can start to
publish messages on the channel. The following sections will provide an example
of message publishing for each of the supported protocols.

## HTTP

To publish message over channel, thing should send following request:

```
curl -s -S -i --cacert docker/ssl/certs/ca.crt -X POST -H "Authorization: <thing_token>" https://localhost/http/channels/<channel_id>/messages -d '[{"bn":"some-base-name:","bt":1.276020076001e+09, "bu":"A","bver":5, "n":"voltage","u":"V","v":120.1}, {"n":"current","t":-5,"v":1.2}, {"n":"current","t":-4,"v":1.3}]'
```

Note that if you're going to use senml message format, you should always send
messages as an array.

For more information about the HTTP messaging service API, please check out the [API documentation](https://github.com/mainflux/mainflux/blob/master/http/openapi.yml).

## MQTT

To send and receive messages over MQTT you could use [Mosquitto tools](https://mosquitto.org),
or [Paho](https://www.eclipse.org/paho/) if you want to use MQTT over WebSocket.

To publish message over channel, thing should call following command:

```
mosquitto_pub -u <thing_id> -P <thing_key> -t channels/<channel_id>/messages -h localhost -m '[{"bn":"some-base-name:","bt":1.276020076001e+09, "bu":"A","bver":5, "n":"voltage","u":"V","v":120.1}, {"n":"current","t":-5,"v":1.2}, {"n":"current","t":-4,"v":1.3}]'
```

To subscribe to channel, thing should call following command:

```
mosquitto_sub -u <thing_id> -P <thing_key> -t channels/<channel_id>/messages -h localhost
```

If you want to use standard topic such as `channels/<channel_id>/messages` with SenML content type (JSON or CBOR), you should use following topic `channels/<channel_id>/messages`.

If you are using TLS to secure MQTT connection, add `--cafile docker/ssl/certs/ca.crt`
to every command.

## CoAP

CoAP adapter implements CoAP protocol using underlying UDP and according to [RFC 7252](https://tools.ietf.org/html/rfc7252). To send and receive messages over CoAP, you can use [Copper](https://github.com/mkovatsc/Copper) CoAP user-agent. To set the add-on, please follow the installation instructions provided [here](https://github.com/mkovatsc/Copper#how-to-integrate-the-copper-sources-into-firefox). Once the Mozilla Firefox and Copper are ready and CoAP adapter is running locally on the default port (5683), you can navigate to the appropriate URL and start using CoAP. The URL should look like this:

```
coap://localhost/channels/<channel_id>/messages?auth=<thing_auth_key>
```

To send a message, use `POST` request.
To subscribe, send `GET` request with Observe option set to 0. There are two ways to unsubscribe:
  1) Send `GET` request with Observe option set to 1.
  2) Forget the token and send `RST` message as a response to `CONF` message received by the server.

The most of the notifications received from the Adapter are non-confirmable. By [RFC 7641](https://tools.ietf.org/html/rfc7641#page-18):

> Server must send a notification in a confirmable message instead of a non-confirmable message at least every 24 hours. This prevents a client that went away or is no longer interested from remaining in the list of observers indefinitely.

CoAP Adapter sends these notifications every 12 hours. To configure this period, please check [adapter documentation](https://www.github.com/mainflux/mainflux/tree/master/coap/README.md) If the client is no longer interested in receiving notifications, the second scenario described above can be used to unsubscribe.

## WS
Mainflux supports [MQTT-over-WS](https://www.hivemq.com/blog/mqtt-essentials-special-mqtt-over-websockets/#:~:text=In%20MQTT%20over%20WebSockets%2C%20the,(WebSockets%20also%20leverage%20TCP).), rather than pure WS protocol. this bring numerous benefits for IoT applications that are derived from the properties of MQTT - like QoS and PUB/SUB features.

There are 2 reccomended Javascript libraries for implementing browser support for Mainflux MQTT-over-WS connectivity:
1. [Eclipse Paho JavaScript Client](https://www.eclipse.org/paho/index.php?page=clients/js/index.php)
2. [MQTT.js](https://github.com/mqttjs/MQTT.js)

As WS is an extension of HTTP protocol, Mainflux exposes it on port `80`, so it's usage is practically transparent.
Additionally, please notice that since same port as for HTTP is used (`80`), and extension URL `/mqtt` should be used -
i.e. connection URL should be `ws://<host_addr>/mqtt`.

> For quick testing you can use [HiveMQ UI tool](http://www.hivemq.com/demos/websocket-client/).

Here is an example of a browser application connecting to Mainflux server and sending and receiving messages over WebSocket using MQTT.js library:

```javascript
<script src="https://unpkg.com/mqtt/dist/mqtt.min.js"></script>
<script>
    // Initialize a mqtt variable globally
    console.log(mqtt)

    // connection option
    const options = {
        clean: true, // retain session
        connectTimeout: 4000, // Timeout period
        // Authentication information
        clientId: '14d6c682-fb5a-4d28-b670-ee565ab5866c',
        username: '14d6c682-fb5a-4d28-b670-ee565ab5866c',
        password: 'ec82f341-d4b5-4c77-ae05-34877a62428f',
    }

    var channelId = '08676a76-101d-439c-b62e-d4bb3b014337'
    var topic = 'channels/' + channelId + '/messages'

    // Connect string, and specify the connection method by the protocol
    // ws Unencrypted WebSocket connection
    // wss Encrypted WebSocket connection
    const connectUrl = 'ws://localhost/mqtt'
    const client = mqtt.connect(connectUrl, options)

    client.on('reconnect', (error) => {
        console.log('reconnecting:', error)
    })

    client.on('error', (error) => {
        console.log('Connection failed:', error)
    })

    client.on('connect', function () {
        console.log('client connected:' + options.clientId)
        client.subscribe(topic, { qos: 0 })
        client.publish(topic, 'WS connection demo!', { qos: 0, retain: false })
    })

    client.on('message', function (topic, message, packet) {
        console.log('Received Message:= ' + message.toString() + '\nOn topic:= ' + topic)
    })

    client.on('close', function () {
        console.log(options.clientId + ' disconnected')
    })
</script>
```

>**N.B.** Eclipse Paho lib adds sub-URL `/mqtt` automaticlly, so procedure for connecting to the server can be something like this:
>```javascript
>var loc = { hostname: 'localhost', port: 80 }
>// Create a client instance
>client = new Paho.MQTT.Client(loc.hostname, Number(loc.port), "clientId")
>// Connect the client
>client.connect({onSuccess:onConnect});
>```

## Subtopics

In order to use subtopics and give more meaning to your pub/sub channel, you can simply add any suffix to base `/channels/<channel_id>/messages` topic.

Example subtopic publish/subscribe for bedroom temperature would be `channels/<channel_id>/messages/bedroom/temperature`.

Subtopics are generic and multilevel. You can use almost any suffix with any depth.

Topics with subtopics are propagated to NATS broker in the following format `channels.<channel_id>.<optional_subtopic>`.

Our example topic `channels/<channel_id>/messages/bedroom/temperature` will be translated to appropriate NATS topic `channels.<channel_id>.bedroom.temperature`.

You can use multilevel subtopics, that have multiple parts. These parts are separated by `.` or `/` separators.
When you use combination of these two, have in mind that behind the scene, `/` separator will be replaced with `.`.
Every empty part of subtopic will be removed. What this means is that subtopic `a///b` is equivalent to `a/b`.
When you want to subscribe, you can use NATS wildcards `*` and `>`. Every subtopic part can have `*` or `>` as it's value, but if there is any other character beside these wildcards, subtopic will be invalid. What this means is that subtopics such as `a.b*c.d` will be invalid, while `a.b.*.c.d` will be valid.

Authorization is done on channel level, so you only have to have access to channel in order to have access to
it's subtopics.

**Note:** When using MQTT, it's recommended that you use standard MQTT wildcards `+` and `#`.

For more information and examples checkout [official nats.io documentation](https://nats.io/documentation/writing_applications/subscribing/)
