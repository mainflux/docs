Bridging with an OPC-UA Server can be done over the [opcua-adapter](https://github.com/mainflux/mainflux/tree/master/opcua). This service sits between Mainflux and an [OPC-UA Server](https://en.wikipedia.org/wiki/OPC_Unified_Architecture) and just forwards the messages from one system to another.

## Run OPC-UA Server

The OPC-UA Server is used for connectivity layer. It allows various methods to read informations from the OPC-UA server and its nodes. The current version of the opcua-adapter still experimental and only `Browse` and `Subscribe` methods are implemented.
[Public OPC-UA test servers](https://github.com/node-opcua/node-opcua/wiki/publicly-available-OPC-UA-Servers-and-Clients) are available for testing of OPC-UA clients and can be used for development and test purposes.

## Mainflux OPC-UA Adapter

Execute the following command from Mainflux project root to run the opcua-adapter:

```bash
docker-compose -f docker/addons/opcua-adapter/docker-compose.yml up -d
```

### Route Map

The opcua-adapter use [Redis](https://redis.io/) database to create a route-map between Mainflux and an OPC-UA Server. As Mainflux use Things and Channels IDs to sign messages, OPC-UA use node ID (node namespace and node identifier combination) and server URI. The adapter route-map associate a `Thing ID` with a `Node ID` and a `Channel ID` with a `Server URI`.

The opcua-adapter uses the matadata of provision events emitted by Mainflux system to update its route map. For that, you must provision Mainflux Channels and Things with an extra metadata key in the JSON Body of the HTTP request. It must be a JSON object with key `opcua` which value is another JSON object. This nested JSON object should contain `node_id` or `server_uri` that correspond to an existent OPC-UA `Node ID` or `Server URI`:

**Channel structure:**

```
{
  "name": "<channel name>",
  "metadata:": {
    "opcua": {
      "server_uri": "<Server URI>"
    }
  }
}
```

**Thing structure:**

```
{
  "name": "<thing name>",
  "metadata:": {
    "opcua": {
      "node_id": "<Node ID>",
    }
  }
}
```

### Browse

The opcua-adapter exposes a `/browse` HTTP endpoint accessible with method `GET` and configurable throw HTTP query parameters `server`, `namespace` and `identifier`. The server URI, the node namespace and the node identifier represent the parent node and are used to fetch the list of available children nodes starting from the given one. By default the root node ID (node namespace and node identifier combination) of an OPC-UA server is `ns=0;i=84`. It's also the default value used by the opcua-adapter to do the browsing if only the server URI is specified in the HTTP query.

### Subscribe

To create an OPC-UA subscription, user should connect the Thing to the Channel. This will automatically create the connection, enable the redis route-map and run a subscription to the `server_uri` and `node_id` defined in the Thing and Channel metadata.

### Messaging

To forward OPC-UA messages the opcua-adapter subscribes to the Node ID of an OPC-UA Server URI. It verifies the `server_uri` and the `node_id` of received messages. If the mapping exists it uses corresponding `Channel ID` and `Thing ID` to sign and forwards the content of the OPC-UA message to the Mainflux message broker. If the mapping or the connection between the Thing and the Channel don't exist the subscription stops.
