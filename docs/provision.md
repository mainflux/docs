# Provision

Provisioning is a process of configuration of an IoT platform in which system operator creates and sets-up different entities used in the platform - users, channels and things.

## Platform management

### Users Management

#### Account Creation

Use the Mainflux API to create user account:

```
curl -s -S -i --cacert docker/ssl/certs/ca.crt -X POST -H "Content-Type: application/json" https://localhost/users -d '{"email":"john.doe@email.com", "password":"12345678"}'
```

Note that when using official `docker-compose`, all services are behind `nginx` proxy and all traffic is `TLS` encrypted.

#### Obtaining an Authorization Key

In order for this user to be able to authenticate to the system, you will have to create an authorization token for him:

```bash
curl -s -S -i --cacert docker/ssl/certs/ca.crt -X POST -H "Content-Type: application/json" https://localhost/tokens -d '{"email":"john.doe@email.com", "password":"12345678"}'
```

Response should look like this:
```json
{
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MjMzODg0NzcsImlhdCI6MTUyMzM1MjQ3NywiaXNzIjoibWFpbmZsdXgiLCJzdWIiOiJqb2huLmRvZUBlbWFpbC5jb20ifQ.cygz9zoqD7Rd8f88hpQNilTCAS1DrLLgLg4PRcH-iAI"
}
```

For more information about the Users service API, please check out the [API documentation](https://github.com/mainflux/mainflux/blob/master/users/openapi.yml).

### System Provisioning

Before proceeding, make sure that you have created a new account and obtained an authorization key.

#### Provisioning Things

> This endpoint will be depreciated in 0.11.0.  It will be replaced with the bulk endpoint currently found at /things/bulk.

Things are created by executing request `POST /things` with a JSON payload. Note that you will also need `user_auth_token` in order to create things that belong to this particular user.

```bash
curl -s -S -i --cacert docker/ssl/certs/ca.crt -X POST -H "Content-Type: application/json" -H "Authorization: <user_auth_token>" https://localhost/things -d '{"name":"weio"}'
```

Response will contain `Location` header whose value represents path to newly created thing:

```
HTTP/1.1 201 Created
Content-Type: application/json
Location: /things/81380742-7116-4f6f-9800-14fe464f6773
Date: Tue, 10 Apr 2018 10:02:59 GMT
Content-Length: 0
```

#### Bulk Provisioning Things

Multiple things can be created by executing a `POST /things/bulk` request with a JSON payload.  The payload should contain a JSON array of the things to be created.  If there is an error any of the things, none of the things will be created.

```bash
curl -s -S -i --cacert docker/ssl/certs/ca.crt -X POST -H "Content-Type: application/json" -H "Authorization: <user_auth_token>" https://localhost/things/bulk -d '[{"name":"weio"},{"name":"bob"}]'
```

The response's body will contain a list of the created things.

```json
HTTP/2 201
server: nginx/1.16.0
date: Tue, 22 Oct 2019 02:19:15 GMT
content-type: application/json
content-length: 222
access-control-expose-headers: Location

{"things":[{"id":"8909adbf-312f-41eb-8cfc-ccc8c4e3655e","name":"weio","key":"4ef103cc-964a-41b5-b75b-b7415c3a3619"},{"id":"2fcd2349-38f7-4b5c-8a29-9607b2ca8ff5","name":"bob","key":"ff0d1490-355c-4dcf-b322-a4c536c8c3bf"}]}
```

#### Retrieving Provisioned Things

In order to retrieve data of provisioned things that is written in database, you can send following request:

```bash
curl -s -S -i --cacert docker/ssl/certs/ca.crt -H "Authorization: <user_auth_token>" https://localhost/things
```

Notice that you will receive only those things that were provisioned by `user_auth_token` owner.

```json
HTTP/1.1 200 OK
Content-Type: application/json
Date: Tue, 10 Apr 2018 10:50:12 GMT
Content-Length: 1105

{
  "total": 2,
  "offset": 0,
  "limit": 10,
  "things": [
    {
      "id": "81380742-7116-4f6f-9800-14fe464f6773",
      "name": "weio",
      "key": "7aa91f7a-cbea-4fed-b427-07e029577590"
    },
    {
      "id": "cb63f852-2d48-44f0-a0cf-e450496c6c92",
      "name": "myapp",
      "key": "cbf02d60-72f2-4180-9f82-2c957db929d1"
    }
  ]
}
```

You can specify `offset` and `limit` parameters in order to fetch a specific group of things. In that case, your request should look like:

```bash
curl -s -S -i --cacert docker/ssl/certs/ca.crt -H "Authorization: <user_auth_token>" https://localhost/things?offset=0&limit=5
```

You can specify `name` and/or `metadata` parameters in order to fetch specific group of things. When specifying metadata you can specify just a part of the metadata JSON you want to match.

```bash
curl -s -S -i --cacert docker/ssl/certs/ca.crt -H "Authorization: <user_auth_token>" https://localhost/things?offset=0&limit=5&metadata="\{\"serial\":\"123456\"\}"
```

If you don't provide them, default values will be used instead: 0 for `offset` and 10 for `limit`. Note that `limit` cannot be set to values greater than 100. Providing invalid values will be considered malformed request.

#### Searching Provisioned Things

In order to search things with specific name and/or metadata, you can send following request:

```bash
curl -s -S -i --cacert docker/ssl/certs/ca.crt -X POST -H "Content-Type: application/json" -H "Authorization: <user_auth_token>" https://localhost/things/search -d '{"metadata":{"foo":"bar"}, "name":"bob", "limit": 10, "offset":0, "order":"name", "dir":"desc"}'
```

You can specify `offset` and `limit` parameters in order to fetch a specific set of things. Also, you can specify ordering with direction through parameters `order` and `dir`. Ordering values can be `name` or `id` of things, order direction can be `asc` or `desc`. If you don't provide them, default values will be used instead: 0 for `offset` and 10 for `limit`. Note that `limit` cannot be set to values greater than 100. Providing invalid values will be considered malformed request.

The response's body will contain a list of the things filtered by name and/or metadata:

```bash
HTTP/2 200 
server: nginx/1.16.0
date: Mon, 15 Mar 2021 18:34:10 GMT
content-type: application/json
content-length: 208
access-control-expose-headers: Location

{
  "total": 1,
  "offset": 0,
  "limit": 10,
  "order": "name",
  "direction": "desc",
  "things": [
    {
      "id": "1b86eea5-94b6-41fa-be9f-d10c85a8994d",
      "name": "bob",
      "key": "d72de10f-4963-4bf1-a454-874a39bb498e",
      "metadata": {
        "foo": "bar"
      }
    }
  ]
}
```

#### Removing Things

In order to remove you own thing you can send following request:

```bash
curl -s -S -i --cacert docker/ssl/certs/ca.crt -X DELETE -H "Authorization: <user_auth_token>" https://localhost/things/<thing_id>
```

#### Provisioning Channels

> This endpoint will be depreciated in 0.11.0.  It will be replaced with the bulk endpoint currently found at /channels/bulk.

Channels are created by executing request `POST /channels`:

```
curl -s -S -i --cacert docker/ssl/certs/ca.crt -X POST -H "Content-Type: application/json" -H "Authorization: <user_auth_token>" https://localhost/channels -d '{"name":"mychan"}'
```

After sending request you should receive response with `Location` header that contains path to newly created channel:

```
HTTP/1.1 201 Created
Content-Type: application/json
Location: /channels/19daa7a8-a489-4571-8714-ef1a214ed914
Date: Tue, 10 Apr 2018 11:30:07 GMT
Content-Length: 0
```

#### Bulk Provisioning Channels

Multiple channels can be created by executing a `POST /things/bulk` request with a JSON payload.  The payload should contain a JSON array of the channels to be created.  If there is an error any of the channels, none of the channels will be created.

```bash
curl -s -S -i --cacert docker/ssl/certs/ca.crt -X POST -H "Content-Type: application/json" -H "Authorization: <user_auth_token>" https://localhost/channels/bulk -d '[{"name":"joe"},{"name":"betty"}]'
```

The response's body will contain a list of the created channels.

```json
HTTP/2 201
server: nginx/1.16.0
date: Tue, 22 Oct 2019 02:14:41 GMT
content-type: application/json
content-length: 135
access-control-expose-headers: Location

{"channels":[{"id":"5a21bbcb-4c9a-4bb4-af31-9982d00f7a6e","name":"joe"},{"id":"d74b119b-2eea-4285-a999-9f747869bb45","name":"betty"}]}
```

#### Retrieving Provisioned Channels

To retreve provisioned channels you should send request to `/channels` with authorization token in `Authorization` header:

```bash
curl -s -S -i --cacert docker/ssl/certs/ca.crt -H "Authorization: <user_auth_token>" https://localhost/channels
```

Note that you will receive only those channels that were created by authorization token's owner.

```json
HTTP/1.1 200 OK
Content-Type: application/json
Date: Tue, 10 Apr 2018 11:38:06 GMT
Content-Length: 139

{
  "total": 1,
  "offset": 0,
  "limit": 10,
  "channels": [
    {
      "id": "19daa7a8-a489-4571-8714-ef1a214ed914",
      "name": "mychan"
    }
  ]
}
```

You can specify  `offset` and  `limit` parameters in order to fetch specific group of channels. In that case, your request should look like:

```bash
curl -s -S -i --cacert docker/ssl/certs/ca.crt -H "Authorization: <user_auth_token>" https://localhost/channels?offset=0&limit=5
```

If you don't provide them, default values will be used instead: 0 for `offset` and 10 for `limit`. Note that `limit` cannot be set to values greater than 100. Providing invalid values will be considered malformed request.

#### Removing Channels

In order to remove specific channel you should send following request:

``` bash
curl -s -S -i --cacert docker/ssl/certs/ca.crt -X DELETE -H "Authorization: <user_auth_token>" https://localhost/channels/<channel_id>
```

### Access Control

Channel can be observed as a communication group of things. Only things that are connected to the channel can send and receive messages from other things in this channel. 
Things that are not connected to this channel are not allowed to communicate over it.

Only user, who is the owner of a channel and of the things, can connect the things to the channel (which is equivalent of giving permissions to these things to communicate over given communication group).

To connect a thing to the channel you should send following request:

> This endpoint will be depreciated in 0.11.0.  It will be replaced with the bulk endpoint found at /connect.

```bash
curl -s -S -i --cacert docker/ssl/certs/ca.crt -X PUT -H "Authorization: <user_auth_token>" https://localhost/channels/<channel_id>/things/<thing_id>
```

To connect multiple things to a channel, you can send the following request:

```bash
curl -s -S -i --cacert docker/ssl/certs/ca.crt -X POST -H "Content-Type: application/json" -H "Authorization: <user_auth_token>" https://localhost/connect -d '{"channel_ids":["<channel_id>", "<channel_id>"],"thing_ids":["<thing_id>", "<thing_id>"]}'
```

You can observe which things are connected to specific channel:

```bash
curl -s -S -i --cacert docker/ssl/certs/ca.crt -H "Authorization: <user_auth_token>" https://localhost/channels/<channel_id>/things
```

You can also observe which things are not connected to specific channel by adding a query parameter `connected=false` to the HTTP request:

```
curl -s -S -i --cacert docker/ssl/certs/ca.crt -H "Authorization: <user_auth_token>" https://localhost/channels/<channel_id>/things?connected=false
```

Response that you'll get should look like this:

```json
{
  "total": 2,
  "offset": 0,
  "limit": 10,
  "things": [
    {
      "id": "3ffb3880-d1e6-4edd-acd9-4294d013f35b",
      "name": "d0",
      "key": "b1996995-237a-4552-94b2-83ec2e92a040",
      "metadata": "{}"
    },
    {
      "id": "94d166d6-6477-43dc-93b7-5c3707dbef1e",
      "name": "d1",
      "key": "e4588a68-6028-4740-9f12-c356796aebe8",
      "metadata": "{}"
    }
  ]
}
```

You can observe to which channels is specified thing connected:

```bash
curl -s -S -i --cacert docker/ssl/certs/ca.crt -H "Authorization: <user_auth_token>" https://localhost/things/<thing_id>/channels
```

You can also observe to which channels is specified thing not connected by adding a query parameter `connected=false` to the HTTP request:

```
curl -s -S -i --cacert docker/ssl/certs/ca.crt -H "Authorization: <user_auth_token>" https://localhost/things/<thing_id>/channels?connected=false
```

Response that you'll get should look like this:

```json
{
  "total": 2,
  "offset": 0,
  "limit": 10,
  "channels": [
    {
      "id": "5e62eb13-2695-4860-8d87-85b8a2f80fd4",
      "name": "c1",
      "metadata": "{}"
    },
    {
      "id": "c4b5e19a-7ffe-4172-b2c5-c8b9d570a165",
      "name": "c0",
      "metadata":"{}"
    }
  ]
}
```

If you want to disconnect your thing from the channel, send following request:

```bash
curl -s -S -i --cacert docker/ssl/certs/ca.crt -X DELETE -H "Authorization: <user_auth_token>" https://localhost/channels/<channel_id>/things/<thing_id>
```

For more information about the Things service API, please check out the [API documentation](https://github.com/mainflux/mainflux/blob/master/things/openapi.yml).

## Provision Service

Provisioning is a process of configuration of an IoT platform in which system operator creates and sets-up different entities used in the platform - users, channels and things. It is part of process of setting up IoT applications where we connect devices on edge with platform in cloud.

For provisioning we can use [Mainflux CLI][cli] for creating users and for each node in the edge (eg. gateway) required number of things, channels, connecting them and creating certificates if needed.

Provision service is used to set up initial application configuration once user is created. Provision service creates  things, channels, connections and certificates. Once user is created we can use provision to create a setup for edge node in one HTTP request instead of issuing several CLI commands.

Provision service provides an HTTP API to interact with [Mainflux][mainflux].

For gateways to communicate with [Mainflux][mainflux] configuration is required (MQTT host, thing, channels, certificates...). Gateway will send a request to [Bootstrap][bootstrap] service providing `<external_id>` and `<external_key>` in HTTP request to get the configuration. To make a request to [Bootstrap][bootstrap] service you can use [Agent][agent] service on a gateway.

To create bootstrap configuration you can use [Bootstrap][bootstrap] or `Provision` service. [Mainflux UI][mfxui] uses [Bootstrap][bootstrap] service for creating gateway configurations. `Provision` service should provide an easy way of provisioning your gateways i.e creating bootstrap configuration and as many things and channels that your setup requires.

Also, you may use provision service to create certificates for each thing. Each service running on gateway may require more than one thing and channel for communication.
If, for example, you are using services [Agent][agent] and [Export][exp] on a gateway you will need two channels for `Agent` (`data` and `control`) and one thing for `Export`.
Additionally, if you enabled mTLS each service will need its own thing and certificate for access to [Mainflux][mainflux].
Your setup could require any number of things and channels, this kind of setup we can call `provision layout`.

Provision service provides a way of specifying this `provision layout` and creating a setup according to that layout by serving requests on `/mapping` endpoint. Provision layout is configured in [config.toml][conftoml].

### Configuration

The service is configured using the environment variables presented in the following [table][config]. Note that any unset variables will be replaced with their default values.


By default, call to `/mapping` endpoint will create one thing and two channels (`control` and `data`) and connect it as this is typical setup required by [Agent](/edge/#agent). If there is a requirement for different provision layout we can use [config][conftoml] file in addition to environment variables.

For the purposes of running provision as an add-on in docker composition environment variables seems more suitable. Environment variables are set in [.env][env].

Configuration can be specified in [config.toml][conftoml]. Config file can specify all the settings that environment variables can configure and in addition `/mapping` endpoint provision layout can be configured.

In `config.toml` we can enlist an array of things and channels that we want to create and make connections between them which we call provision layout.

Things Metadata can be whatever suits your needs. Thing that has metadata with `external_id` will have bootstrap configuration created, `external_id` value will be populated with value from [request](#example)).
Bootstrap configuration can be fetched with [Agent][agent]. For channel's metadata `type` is reserved for `control` and `data` which we use with [Agent][agent].

Example of provision layout below
```toml
[bootstrap]
  [bootstrap.content]
    [bootstrap.content.agent.edgex]
      url = "http://localhost:48090/api/v1/"

    [bootstrap.content.agent.log]
      level = "info"

    [bootstrap.content.agent.mqtt]
      mtls = false
      qos = 0
      retain = false
      skip_tls_ver = true
      url = "localhost:1883"

    [bootstrap.content.agent.server]
      nats_url = "localhost:4222"
      port = "9000"

    [bootstrap.content.agent.heartbeat]
      interval = "30s"

    [bootstrap.content.agent.terminal]
      session_timeout = "30s"

    [bootstrap.content.export.exp]
      log_level = "debug"
      nats = "nats://localhost:4222"
      port = "8172"
      cache_url = "localhost:6379"
      cache_pass = ""
      cache_db = "0"

    [bootstrap.content.export.mqtt]
      ca_path = "ca.crt"
      cert_path = "thing.crt"
      channel = ""
      host = "tcp://localhost:1883"
      mtls = false
      password = ""
      priv_key_path = "thing.key"
      qos = 0
      retain = false
      skip_tls_ver = false
      username = ""

    [[bootstrap.content.export.routes]]
      mqtt_topic = ""
      nats_topic = "channels"
      subtopic = ""
      type = "mfx"
      workers = 10

    [[bootstrap.content.export.routes]]
      mqtt_topic = ""
      nats_topic = "export"
      subtopic = ""
      type = "default"
      workers = 10

[[things]]
  name = "thing"

  [things.metadata]
    external_id = "xxxxxx"

[[channels]]
  name = "control-channel"

  [channels.metadata]
    type = "control"

[[channels]]
  name = "data-channel"

  [channels.metadata]
    type = "data"

[[channels]]
  name = "export-channel"

  [channels.metadata]
    type = "export"
```

`[bootstrap.content]` will be marshalled and saved into `content` field in bootstrap configs when request to `/mappings` is made, `content` field from bootstrap config is used to create `Agent` and `Export` configuration files upon `Agent` fetching bootstrap configuration.

### Authentication
In order to create necessary entities provision service needs to authenticate against Mainflux.
To provide authentication credentials to the provision service you can pass it in as an environment variable or in a config file as Mainflux user and password or as API token (that can be issued on `/users` or `/keys` endpoint of [auth][auth].

Additionally, users or API token can be passed in Authorization header, this authentication takes precedence over others.

* `username`, `password` - (`MF_PROVISION_USER`, `MF_PROVISION_PASSWORD` in [.env][env], `mf_user`, `mf_pass` in [config.toml][conftoml]
* API Key - (`MF_PROVISION_API_KEY` in [.env][env] or [config.toml][conftoml]
* `Authorization: Token|ApiKey` - request authorization header containing either users token or API key. Check [auth][auth].

### Running
Provision service can be run as a standalone or in docker composition as addon to the core docker composition.

Standalone:
```bash
MF_PROVISION_BS_SVC_URL=http://localhost:8202/things \
MF_PROVISION_THINGS_LOCATION=http://localhost:8182 \
MF_PROVISION_USERS_LOCATION=http://localhost:8180 \
MF_PROVISION_CONFIG_FILE=docker/addons/provision/configs/config.toml \
build/mainflux-provision
```

Docker composition:
```bash
docker-compose -f docker/addons/provision/docker-compose.yml up
```

### Provision

For the case that credentials or API token is passed in configuration file or environment variables, call to `/mapping` endpoint doesn't require `Authentication` header:
```bash
curl -s -S  -X POST  http://localhost:8888/mapping  -H 'Content-Type: application/json' -d '{"external_id": "33:52:77:99:43", "external_key": "223334fw2"}'
```

In the case that provision service is not deployed with credentials or API key or you want to use user other than one being set in environment (or config file):
```bash
curl -s -S  -X POST  http://localhost:8091/mapping -H "Authorization: <token|api_key>" -H 'Content-Type: application/json' -d '{"external_id": "<external_id>", "external_key": "<external_key>"}'
```

Or if you want to specify a name for thing different than in `config.toml` you can specify post data as:

```json
{"name": "<name>", "external_id": "<external_id>", "external_key": "<external_key>"}
```

Response contains created things, channels and certificates if any:
```json
{
  "things": [
    {
      "id": "c22b0c0f-8c03-40da-a06b-37ed3a72c8d1",
      "name": "thing",
      "key": "007cce56-e0eb-40d6-b2b9-ed348a97d1eb",
      "metadata": {
        "external_id": "33:52:79:C3:43"
      }
    }
  ],
  "channels": [
    {
      "id": "064c680e-181b-4b58-975e-6983313a5170",
      "name": "control-channel",
      "metadata": {
        "type": "control"
      }
    },
    {
      "id": "579da92d-6078-4801-a18a-dd1cfa2aa44f",
      "name": "data-channel",
      "metadata": {
        "type": "data"
      }
    }
  ],
  "whitelisted": {
    "c22b0c0f-8c03-40da-a06b-37ed3a72c8d1": true
  }
}
```

### Example

Deploy Mainflux UI docker composition as it contains all the required services for provisioning to work ( `certs`, `bootstrap` and Mainflux core)

```
git clone https://github.com/mainflux/ui
cd ui
docker-compose -f docker/docker-compose.yml up
```
Create user and obtain access token

```bash
mainflux-cli -m https://mainflux.com users create john.doe@email.com 12345678

# Retrieve token
mainflux-cli -m https://mainflux.com users token john.doe@email.com 12345678

created: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1OTY1ODU3MDUsImlhdCI6MTU5NjU0OTcwNSwiaXNzIjoibWFpbmZsdXguYXV0aG4iLCJzdWIiOiJtaXJrYXNoQGdtYWlsLmNvbSIsInR5cGUiOjB9._vq0zJzFc9tQqc8x74kpn7dXYefUtG9IB0Cb-X2KMK8
```
Put a value of token into environment variable

```
TOK=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1OTY1ODU3MDUsImlhdCI6MTU5NjU0OTcwNSwiaXNzIjoibWFpbmZsdXguYXV0aG4iLCJzdWIiOiJtaXJrYXNoQGdtYWlsLmNvbSIsInR5cGUiOjB9._vq0zJzFc9tQqc8x74kpn7dXYefUtG9IB0Cb-X2KMK8
```

Make a call to provision endpoint

```
curl -s -S  -X POST  http://mainflux.com:8190/mapping -H "Authorization: $TOK" -H 'Content-Type: application/json'   -d '{"name":"edge-gw",  "external_id" : "gateway", "external_key":"external_key" }'
```

To check the results you can make a call to bootstrap endpoint

```
curl -s -S -X GET http://mainflux.com:8202/things/bootstrap/gateway -H "Authorization: external_key" -H 'Content-Type: application/json'
```

Or you can start `Agent` with:

```bash
git clone https://github.com/mainflux/agent
cd agent
make
MF_AGENT_BOOTSTRAP_ID=gateway MF_AGENT_BOOTSTRAP_KEY=external_key MF_AGENT_BOOTSTRAP_URL=http://mainflux.ccom:8202/things/bootstrap build/mainflux-agent
```

Agent will retrieve connections parameters and connect to Mainflux cloud.






[mainflux]: https://github.com/mainflux/mainflux
[bootstrap]: https://github.com/mainflux/mainflux/tree/master/bootstrap
[export]: https://github.com/mainflux/export
[agent]: https://github.com/mainflux/agent
[mfxui]: https://github.com/mainflux/mainflux/ui
[config]: https://github.com/mainflux/mainflux/tree/master/provision#configuration
[env]: https://github.com/mainflux/mainflux/blob/master/.env
[conftoml]: https://github.com/mainflux/mainflux/blob/master/docker/addons/provision/configs/config.toml
[auth]: https://github.com/mainflux/mainflux/blob/master/auth/README.md
[exp]: https://github.com/mainflux/export
[cli]: https://github.com/mainflux/mainflux/tree/master/cli

For more information about the Provision service API, please check out the [API documentation](https://github.com/mainflux/mainflux/blob/master/provision/openapi.yml).

## Certs Service
Issues certificates for things. `Certs` service can create certificates to be used when `Mainflux` is deployed to support mTLS.  
`Certs` service will create certificate for valid thing ID if valid user token is passed and user is owner of the provided thing ID.

Certificate service can create certificates in two modes:
1. Development mode - to be used when no PKI is deployed, this works similar to the [make thing_cert](../docker/ssl/Makefile)
2. PKI mode - certificates issued by PKI, when you deploy `Vault` as PKI certificate management `cert` service will proxy requests to `Vault` previously checking access rights and saving info on successfully created certificate. 



### Development mode
If `MF_CERTS_VAULT_HOST` is empty than Development mode is on.

To issue a certificate:
```bash

TOK=`curl  -s --insecure -S -X POST http://localhost/tokens -H 'Content-Type: application/json' -d '{"email":"edge@email.com","password":"12345678"}' | jq -r '.token'`

curl -s -S  -X POST  http://localhost:8204/certs -H "Authorization: $TOK" -H 'Content-Type: application/json'   -d '{"thing_id":<thing_id>, "rsa_bits":2048, "key_type":"rsa"}'
```

```json
{
  "ThingID": "",
  "ClientCert": "-----BEGIN CERTIFICATE-----\nMIIDmTCCAoGgAwIBAgIRANmkAPbTR1UYeYO0Id/4+8gwDQYJKoZIhvcNAQELBQAw\nVzESMBAGA1UEAwwJbG9jYWxob3N0MREwDwYDVQQKDAhNYWluZmx1eDEMMAoGA1UE\nCwwDSW9UMSAwHgYJKoZIhvcNAQkBFhFpbmZvQG1haW5mbHV4LmNvbTAeFw0yMDA2\nMzAxNDIxMDlaFw0yMDA5MjMyMjIxMDlaMFUxETAPBgNVBAoTCE1haW5mbHV4MREw\nDwYDVQQLEwhtYWluZmx1eDEtMCsGA1UEAxMkYjAwZDBhNzktYjQ2YS00NTk3LTli\nNGYtMjhkZGJhNTBjYTYyMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA\ntgS2fLUWG3CCQz/l6VRQRJfRvWmdxK0mW6zIXGeeOILYZeaLiuiUnohwMJ4RiMqT\nuJbInAIuO/Tt5osfrCFFzPEOLYJ5nZBBaJfTIAxqf84Ou1oeMRll4wpzgeKx0rJO\nXMAARwn1bT9n3uky5QQGSLy4PyyILzSXH/1yCQQctdQB/Ar/UI1TaYoYlGzh7dHT\nWpcxq1HYgCyAtcrQrGD0rEwUn82UBCrnya+bygNqu0oDzIFQwa1G8jxSgXk0mFS1\nWrk7rBipsvp8HQhdnvbEVz4k4AAKcQxesH4DkRx/EXmU2UvN3XysvcJ2bL+UzMNI\njNhAe0pgPbB82F6zkYZ/XQIDAQABo2IwYDAOBgNVHQ8BAf8EBAMCB4AwHQYDVR0l\nBBYwFAYIKwYBBQUHAwIGCCsGAQUFBwMBMA4GA1UdDgQHBAUBAgMEBjAfBgNVHSME\nGDAWgBRs4xR91qEjNRGmw391xS7x6Tc+8jANBgkqhkiG9w0BAQsFAAOCAQEAW/dS\nV4vNLTZwBnPVHUX35pRFxPKvscY+vnnpgyDtITgZHYe0KL+Bs3IHuywtqaezU5x1\nkZo+frE1OcpRvp7HJtDiT06yz+18qOYZMappCWCeAFWtZkMhlvnm3TqTkgui6Xgl\nGj5xnPb15AOlsDE2dkv5S6kEwJGHdVX6AOWfB4ubUq5S9e4ABYzXGUty6Hw/ZUmJ\nhCTRVJ7cQJVTJsl1o7CYT8JBvUUG75LirtoFE4M4JwsfsKZXzrQffTf1ynqI3dN/\nHWySEbvTSWcRcA3MSmOTxGt5/zwCglHDlWPKMrXtjTW7NPuGL5/P9HSB9HGVVeET\nDUMdvYwgj0cUCEu3LA==\n-----END CERTIFICATE-----\n",
  "IssuingCA": "",
  "CAChain": null,
  "ClientKey": "-----BEGIN RSA PRIVATE KEY-----\nMIIEowIBAAKCAQEAtgS2fLUWG3CCQz/l6VRQRJfRvWmdxK0mW6zIXGeeOILYZeaL\niuiUnohwMJ4RiMqTuJbInAIuO/Tt5osfrCFFzPEOLYJ5nZBBaJfTIAxqf84Ou1oe\nMRll4wpzgeKx0rJOXMAARwn1bT9n3uky5QQGSLy4PyyILzSXH/1yCQQctdQB/Ar/\nUI1TaYoYlGzh7dHTWpcxq1HYgCyAtcrQrGD0rEwUn82UBCrnya+bygNqu0oDzIFQ\nwa1G8jxSgXk0mFS1Wrk7rBipsvp8HQhdnvbEVz4k4AAKcQxesH4DkRx/EXmU2UvN\n3XysvcJ2bL+UzMNIjNhAe0pgPbB82F6zkYZ/XQIDAQABAoIBAALoal3tqq+/iWU3\npR2oKiweXMxw3oNg3McEKKNJSH7QoFJob3xFoPIzbc9pBxCvY9LEHepYIpL0o8RW\nHqhqU6olg7t4ZSb+Qf1Ax6+wYxctnJCjrO3N4RHSfevqSjr6fEQBEUARSal4JNmr\n0hNUkCEjWrIvrPFMHsn1C5hXR3okJQpGsad4oCGZDp2eZ/NDyvmLBLci9/5CJdRv\n6roOF5ShWweKcz1+pfy666Q8RiUI7H1zXjPaL4yqkv8eg/WPOO0dYF2Ri2Grk9OY\n1qTM0W1vi9zfncinZ0DpgtwMTFQezGwhUyJHSYHmjVBA4AaYIyOQAI/2dl5fXM+O\n9JfXpOUCgYEA10xAtMc/8KOLbHCprpc4pbtOqfchq/M04qPKxQNAjqvLodrWZZgF\nexa+B3eWWn5MxmQMx18AjBCPwbNDK8Rkd9VqzdWempaSblgZ7y1a0rRNTXzN5DFP\noiuRQV4wszCuj5XSdPn+lxApaI/4+TQ0oweIZCpGW39XKePPoB5WZiMCgYEA2G3W\niJncRpmxWwrRPi1W26E9tWOT5s9wYgXWMc+PAVUd/qdDRuMBHpu861Qoghp/MJog\nBYqt2rQqU0OxvIXlXPrXPHXrCLOFwybRCBVREZrg4BZNnjyDTLOu9C+0M3J9ImCh\n3vniYqb7S0gRmoDM0R3Zu4+ajfP2QOGLXw1qHH8CgYEAl0EQ7HBW8V5UYzi7XNcM\nixKOb0YZt83DR74+hC6GujTjeLBfkzw8DX+qvWA8lxLIKVC80YxivAQemryv4h21\nX6Llx/nd1UkXUsI+ZhP9DK5y6I9XroseIRZuk/fyStFWsbVWB6xiOgq2rKkJBzqw\nCCEQpx40E6/gsqNDiIAHvvUCgYBkkjXc6FJ55DWMLuyozfzMtpKsVYeG++InSrsM\nDn1PizQS/7q9mAMPLCOP312rh5CPDy/OI3FCbfI1GwHerwG0QUP/bnQ3aOTBmKoN\n7YnsemIA/5w16bzBycWE5x3/wjXv4aOWr9vJJ/siMm0rtKp4ijyBcevKBxHpeGWB\nWAR1FQKBgGIqAxGnBpip9E24gH894BaGHHMpQCwAxARev6sHKUy27eFUd6ipoTva\n4Wv36iz3gxU4R5B0gyfnxBNiUab/z90cb5+6+FYO13kqjxRRZWffohk5nHlmFN9K\nea7KQHTfTdRhOLUzW2yVqLi9pzfTfA6Yqf3U1YD3bgnWrp1VQnjo\n-----END RSA PRIVATE KEY-----\n",
  "PrivateKeyType": "",
  "Serial": "",
  "Expire": "0001-01-01T00:00:00Z"
}
```

### PKI mode

When `MF_CERTS_VAULT_HOST` is set it is presumed that `Vault` is installed and `certs` service will issue certificates using `Vault` API.
First you'll need to set up `Vault`. 
To setup `Vault` follow steps in [Build Your Own Certificate Authority (CA)](https://learn.hashicorp.com/tutorials/vault/pki-engine).

To setup certs service with `Vault` following environment variables must be set:

```
MF_CERTS_VAULT_HOST=vault-domain.com
MF_CERTS_VAULT_PKI_PATH=<vault_pki_path>
MF_CERTS_VAULT_ROLE=<vault_role>
MF_CERTS_VAULT_TOKEN=<vault_acces_token>
```

For lab purposes you can use docker-compose and script for setting up PKI in [https://github.com/mteodor/vault](https://github.com/mteodor/vault)

Issuing certificate is same as in **Development** mode.
In this mode certificates can also be revoked:

```bash
curl -s -S -X DELETE http://localhost:8204/certs/revoke -H "Authorization: $TOK" -H 'Content-Type: application/json'   -d '{"thing_id":"c30b8842-507c-4bcd-973c-74008cef3be5"}'
```

For more information about the Certification service API, please check out the [API documentation](https://github.com/mainflux/mainflux/blob/master/certs/openapi.yml).