# Edge

Mainflux IoT platform provides services for supporting management of devices on the edge. Typically, IoT solution includes devices (sensors/actuators) deployed in far edge and connected through some proxy gateway. 
Although most devices could be connected to the Mainflux directly, using gateways decentralizes system, decreases load on the cloud and makes setup less difficult. Also, gateways can provide additional data processing, filtering and storage.

Services that can be used on gateway to enable data and control plane for edge:

* [Agent](/edge/#agent)
* [Export](/edge/#export)
* [Mainflux](/architecture/)

![Edge1](img/edge/edge.png)
*Fig. 1*


Figure shows edge gateway that is running Agent, Export and minimal deployment of Mainflux services.
Mainflux services enable device management and MQTT protocol, NATS being a central message bus in Mainflux becomes also central message bus for other services like `Agent` and `Export` as well as for any new custom developed service that can be built to interface with devices with any of hardware supported interfaces on the gateway, those services would publish data to NATS where `Export` service can pick them up and send to cloud.

Agent can be used to control deployed services as well as to monitor their liveliness through subcribing to `heartbeat` NATS subject where services should publish their liveliness status, like `Export` service does.

## Agent

Agent is service that is used to manage gateways that are connected to Mainflux in cloud. It provides a way to send commands to gateway and receive response via mqtt.
There are two types of channels used for **Agent** `data` and `control`. Over the `control` we are sending commands and receiving response from commands.
Data collected from sensors connected to gateway are being sent over `data` channel. Agent is able to configure itself provided that [bootstrap server](/bootstrap/) is running, it will retrieve configuration from bootstrap server provided few arguments - `external_id` and `external_key` see [bootstraping](/bootstrap/#bootstrapping).

Agent service has following features:
* Remote execution of commands
* Remote terminal, remote session to `bash` managed by `Agent`
* Heartbeat - listening to NATS topic `heartbeat.>` it can remotely provide info on running services, if services are publishing heartbeat ( like [Export](/edge/#export))
* Proxying commands to other gateway services
* Edgex SMA - remotely making requests to EdgeX endpoints and fetching results, if EdgeX is deployed.


### Run Agent

Before running agent we need to provision a thing and DATA and CONTROL channel. Thing that will be used as gateway representation and make bootstrap configuration.
If using Mainflux UI this is done automatically when adding gateway through UI.
Gateway can be provisioned with [`provision`](/provision/) service.

When you provisioned gateway as described in [provision](/provision/) you can check results

```bash
curl -s -S -X GET http://mainflux-domain.com:8202/things/bootstrap/<external_id> -H "Authorization: <external_key>" -H 'Content-Type: application/json' |jq
```

```json
{
  "mainflux_id": "e22c383a-d2ab-47c1-89cd-903955da993d",
  "mainflux_key": "fc987711-1828-461b-aa4b-16d5b2c642fe",
  "mainflux_channels": [
    {
      "id": "fa5f9ba8-a1fc-4380-9edb-d0c23eaa24ec",
      "name": "control-channel",
      "metadata": {
        "type": "control"
      }
    },
    {
      "id": "24e5473e-3cbe-43d9-8a8b-a725ff918c0e",
      "name": "data-channel",
      "metadata": {
        "type": "data"
      }
    },
    {
      "id": "1eac45c2-0f72-4089-b255-ebd2e5732bbb",
      "name": "export-channel",
      "metadata": {
        "type": "export"
      }
    }
  ],
  "content": "{\"agent\":{\"edgex\":{\"url\":\"http://localhost:48090/api/v1/\"},\"heartbeat\":{\"interval\":\"30s\"},\"log\":{\"level\":\"debug\"},\"mqtt\":{\"mtls\":false,\"qos\":0,\"retain\":false,\"skip_tls_ver\":true,\"url\":\"tcp://mainflux-domain.com:1883\"},\"server\":{\"nats_url\":\"localhost:4222\",\"port\":\"9000\"},\"terminal\":{\"session_timeout\":\"30s\"}},\"export\":{\"exp\":{\"cache_db\":\"0\",\"cache_pass\":\"\",\"cache_url\":\"localhost:6379\",\"log_level\":\"debug\",\"nats\":\"nats://localhost:4222\",\"port\":\"8172\"},\"mqtt\":{\"ca_path\":\"ca.crt\",\"cert_path\":\"thing.crt\",\"channel\":\"\",\"host\":\"tcp://mainflux-domain.com:1883\",\"mtls\":false,\"password\":\"\",\"priv_key_path\":\"thing.key\",\"qos\":0,\"retain\":false,\"skip_tls_ver\":false,\"username\":\"\"},\"routes\":[{\"mqtt_topic\":\"\",\"nats_topic\":\"channels\",\"subtopic\":\"\",\"type\":\"mfx\",\"workers\":10},{\"mqtt_topic\":\"\",\"nats_topic\":\"export\",\"subtopic\":\"\",\"type\":\"default\",\"workers\":10}]}}"
}
```


- `external_id` is usually MAC address, but anything that suits applications requirements can be used
- `external_key` is key that will be provided to agent process
- `thing_id` is mainflux thing id 
- `channels` is 2-element array where first channel is CONTROL and second is DATA, both channels should be assigned to thing
- `content` is used for configuring parameters of agent and export service.


Then to start the agent service you can do it like this

```
git clone https://github.com/mainflux/agent
make
cd build

MF_AGENT_LOG_LEVEL=debug \
MF_AGENT_BOOTSTRAP_KEY=edged \
MF_AGENT_BOOTSTRAP_ID=34:e1:2d:e6:cf:03 ./mainflux-agent

{"level":"info","message":"Requesting config for 34:e1:2d:e6:cf:03 from http://localhost:8202/things/bootstrap","ts":"2019-12-05T04:47:24.98411512Z"}
{"level":"info","message":"Getting config for 34:e1:2d:e6:cf:03 from http://localhost:8202/things/bootstrap succeeded","ts":"2019-12-05T04:47:24.995465239Z"}
{"level":"info","message":"Connected to MQTT broker","ts":"2019-12-05T04:47:25.009645082Z"}
{"level":"info","message":"Agent service started, exposed port 9000","ts":"2019-12-05T04:47:25.009755345Z"}
{"level":"info","message":"Subscribed to MQTT broker","ts":"2019-12-05T04:47:25.012930443Z"}
```

 - `MF_AGENT_BOOTSTRAP_KEY` - is `external_key` in bootstrap configuration.
 - `MF_AGENT_BOOSTRAP_ID` - is `external_id` in bootstrap configuration.

#### Remote execution of commands via Agent

```bash
# Set connection parameters as environment variables in shell
CH=`curl -s -S -X GET http://some-domain-name:8202/things/bootstrap/34:e1:2d:e6:cf:03 -H "Authorization: edged" -H 'Content-Type: application/json' | jq -r '.mainflux_channels[0].id'`
TH=`curl -s  -S -X GET http://some-domain-name:8202/things/bootstrap/34:e1:2d:e6:cf:03 -H "Authorization: edged" -H 'Content-Type: application/json' | jq -r .mainflux_id`
KEY=`curl -s  -S -X GET http://some-domain-name:8202/things/bootstrap/34:e1:2d:e6:cf:03 -H "Authorization: edged" -H 'Content-Type: application/json' | jq -r .mainflux_key`

# Subscribe for response
mosquitto_sub -d -u $TH -P $KEY  -t channels/$CH/messages/res/# -h some-domain-name -p 1883

# Publish command e.g `ls`
mosquitto_pub -d -u $TH -P $KEY  -t channels/$CH/messages/req -h some-domain-name -p 1883  -m '[{"bn":"1:", "n":"exec", "vs":"ls, -l"}]'
```

#### Remote terminal 

This can be checked from the UI, click on the details for gateway and below the gateway parameters you will se box with prompt, if `agent` is running and it is properly connected you should be able to execute commands remotely.

#### Heartbeat 
If there are services that are running on same gateway as `agent` and they are publishing heartbeat to NATS subject `heartbeat.service_name.service`
You can get the list of services by sending following mqtt message

```
# View services that are sending heartbeat
mosquitto_pub -d -u $TH -P $KEY  -t channels/$CH/messages/req -h some-domain-name -p 1883  -m '[{"bn":"1:", "n":"service", "vs":"view"}]'
```
Response can be observed on `channels/$CH/messages/res/#`

### Proxying commands

You can send commands to services running on the same edge gateway as Agent if they are subscribed on same NATS server and correct subject.

Service commands are being sent via MQTT to topic:

`channels/<control_channel_id>/messages/services/<service_name>/<subtopic>`
  
when messages is received Agent forwards them to NATS on subject:

`commands.<service_name>.<subtopic>`

Payload is up to the application and service itself.


### EdgeX 

[Edgex](https://github.com/edgexfoundry/edgex-go) control messages are sent and received over control channel. MF sends a control SenML of the following form:
```
[{"bn":"<uuid>:", "n":"control", "vs":"<cmd>, <param>, edgexsvc1, edgexsvc2, …, edgexsvcN"}}]
```
For example,
```
[{"bn":"1:", "n":"control", "vs":"operation, stop, edgex-support-notifications, edgex-core-data"}]
```
Agent, on the other hand, returns a response SenML of the following form:
```
[{"bn":"<uuid>:", "n":"<>", "v":"<RESP>"}]
```
### Remote Commands
EdgeX defines SMA commands in the following [RAML file](https://github.com/edgexfoundry/edgex-go/blob/master/api/raml/system-agent.raml)

Commands are:

* OPERATION
* CONFIG
* METRICS
* PING

**Operation**
  
```
mosquitto_pub -u <thing_id> -P <thing_key> -t channels/<channel_id>/messages/req -h localhost -m '[{"bn":"1:", "n":"control", "vs":"edgex-operation, start, edgex-support-notifications, edgex-core-data"}]'
```
**Config**
```
mosquitto_pub -u <thing_id> -P <thing_key> -t channels/<channel_id>/messages/req -h localhost -m '[{"bn":"1:", "n":"control", "vs":"edgex-config, edgex-support-notifications, edgex-core-data"}]'
```
**Metrics**
```
mosquitto_pub -u <thing_id> -P <thing_key> -t channels/<channel_id>/messages/req -h localhost -m '[{"bn":"1:", "n":"control", "vs":"edgex-metrics, edgex-support-notifications, edgex-core-data"}]'
```

If you subscribe to

```
mosquitto_sub -u <thing_id> -P <thing_key> -t channels/<channel_id>/messages/#
```
You can observe commands and response from commands executed against edgex
```
[{"bn":"1:", "n":"control", "vs":"edgex-metrics, edgex-support-notifications, edgex-core-data"}]                                                                            
[{"bn":"1","n":"edgex-metrics","vs":"{\"Metrics\":{\"edgex-core-data\":{\"CpuBusyAvg\":15.568632467698606,\"Memory\":{\"Alloc\":2040136,\"Frees\":876344,\"LiveObjects\":15134,\"Mallocs\":891478,\"Sys\":73332984,\"TotalAlloc\":80657464}},\"edgex-support-notifications\":{\"CpuBusyAvg\":14.65381169745318,\"Memory\":{\"Alloc\":961784,\"Frees\":127430,\"LiveObjects\":6095,\"Mallocs\":133525,\"Sys\":72808696,\"TotalAlloc\":11665416}}}}\n"}] 
```

## Export
Mainflux Export service can send message from one Mainflux cloud to another via MQTT, or it can send messages from edge gateway to Mainflux Cloud.
Export service is subscribed to local message bus and connected to MQTT broker in the cloud.  
Messages collected on local message bus are redirected to the cloud.
When connection is lost, if QoS2 is used, messages from the local bus are stored into file or in memory to be resent upon reconnection.
Additonaly `Export` service publishes liveliness status to `Agent` via NATS subject `heartbeat.export.service`


### Install
Get the code:

```bash
go get github.com/mainflux/export
cd $GOPATH/github.com/mainflux/export
```

Make:
```bash
make
```

### Usage

```bash
cd build
./mainflux-export
```

### Configuration

By default `Export` service looks for config file at [`../configs/config.toml`][conftoml] if no env vars are specified.  

```toml
[exp]
  log_level = "debug"
  nats = "localhost:4222"
  port = "8170"

[mqtt]
  username = "<thing_id>"
  password = "<thing_password>"
  ca_path = "ca.crt"
  client_cert = ""
  client_cert_key = ""
  client_cert_path = "thing.crt"
  client_priv_key_path = "thing.key"
  mtls = "false"
  priv_key = "thing.key"
  retain = "false"
  skip_tls_ver = "false"
  url = "tcp://mainflux.com:1883"

[[routes]]
  mqtt_topic = "channel/<channel_id>/messages"
  subtopic = "subtopic"
  nats_topic = "export"
  type = "default"
  workers = 10

[[routes]]
  mqtt_topic = "channel/<channel_id>/messages"
  subtopic = "subtopic"
  nats_topic = "channels"
  type = "mfx"
  workers = 10
```

### Environment variables

Service will first look for `MF_EXPORT_CONFIG_FILE` for configuration and if not found it will be configured with env variables and new config file specified with `MF_EXPORT_CONFIG_FILE` (default value will be used if none specified) will be saved with values populated from env vars.  
The service is configured using the [environment variables](env) as presented in the table. Note that any unset variables will be replaced with their default values.

For values in environment variables to take effect make sure that there is no `MF_EXPORT_CONFIG_FILE` file.

If you run with environment variables you can create config file:
```bash
MF_EXPORT_PORT=8178 \
MF_EXPORT_LOG_LEVEL=debug \
MF_EXPORT_MQTT_HOST=tcp://localhost:1883 \
MF_EXPORT_MQTT_USERNAME=<thing_id> \
MF_EXPORT_MQTT_PASSWORD=<thing_key> \
MF_EXPORT_MQTT_CHANNEL=<channel_id> \
MF_EXPORT_MQTT_SKIP_TLS=true \
MF_EXPORT_MQTT_MTLS=false \
MF_EXPORT_MQTT_CA=ca.crt \
MF_EXPORT_MQTT_CLIENT_CERT=thing.crt \
MF_EXPORT_MQTT_CLIENT_PK=thing.key \
MF_EXPORT_CONFIG_FILE=export.toml \
../build/mainflux-export&
```
Values from environment variables will be used to populate export.toml

#### Http port

- `port` - HTTP port where status of `Export` service can be fetched.
```bash
curl -X GET http://localhost:8170/version
{"service":"export","version":"0.0.1"}%
``` 

#### MQTT connection

To establish connection to MQTT broker following settings are needed:
- `username` - Mainflux <thing_id>
- `password` - Mainflux <thing_key>
- `url` - url of MQTT broker

Additionally, you will need MQTT client certificates if you enable mTLS. To obtain certificates `ca.crt`, `thing.crt` and key `thing.key` follow instructions [here](https://mainflux.readthedocs.io/en/latest/authentication/#mutual-tls-authentication-with-x509-certificates) or [here](/provision/#certs-service).

#### MTLS

To setup `MTLS` connection `Export` service requires client certificate and `mtls` in config or `MF_EXPORT_MQTT_MTLS` must be set to `true`. 
Client certificate can be provided in a file, `client_cert_path` and `client_cert_key_path` are used for specifying path to certificate files.
If MTLS is used and no certificate file paths are specified then `Export` will look in `client_cert` and `client_cert_key` of config file expecting certificate content stored as string.

#### Routes 

Routes are being used for specifying which subscriber's topic(subject) goes to which publishing topic.
Currently only MQTT is supported for publishing.
To match Mainflux requirements `mqtt_topic` must contain `channel/<channel_id>/messages`, additional subtopics can be appended.

- `mqtt_topic` - `channel/<channel_id>/messages/<custom_subtopic>`
- `nats_topic` - `Export` service will be subscribed to NATS subject `<nats_topic>.>`
- `subtopic` - messages will be published to MQTT topic `<mqtt_topic>/<subtopic>/<nats_subject>`, where dots in nats_subject are replaced with '/'
- `workers` - specifies number of workers that will be used for message forwarding.
- `type` - specifies message transformation:
    - `default` is for sending messages as they are received on NATS with no transformation (so they should be in SenML or JSON format if we want to persist them in Mainflux in cloud). If you don't want to persist messages in Mainflux or you are not exporting to Mainflux cloud - message format can be anything that suits your application as message passes untransformed.
    - `mfx` is for messages that are being picked up on internal Mainflux NATS bus. When using `Export` along with Mainflux deployed on gateway ([Fig. 1](#edge)) messages coming from MQTT broker that are published to NATS bus are [Mainflux message](https://github.com/mainflux/mainflux/blob/master/pkg/messaging/message.proto). Using `mfx` type will extract payload and `export` will publish it to `mqtt_topic`. Extracted payload is SenML or JSON if we want to persist messages. `nats_topic` in this case must be `channels`, or if you want to pick messages from a specific channel in local Mainflux instance to be exported to cloud you can put `channels.<local_mainflux_channel_id>`.

Before running `Export` service edit `configs/config.toml` and provide `username`, `password` and `url`
 * `username` - matches `thing_id` in Mainflux cloud instance
 * `password` - matches `thing_key`
 * `channel` - MQTT part of the topic where to publish MQTT data (`channel/<channel_id>/messages` is format of mainflux MQTT topic) and plays a part in authorization.

If Mainflux and Export service are deployed on same gateway `Export` can be configured to send messages from Mainflux internal NATS bus to Mainflux in a cloud.
In order for `Export` service to listen on Mainflux NATS deployed on the same machine NATS port must be exposed.
Edit Mainflux [docker-compose.yml][docker-compose]. NATS section must look like below:
```
  nats:
    image: nats:1.3.0
    container_name: mainflux-nats
    restart: on-failure
    networks:
      - mainflux-base-net
    ports:
      - 4222:4222
```
  
### How to save config via agent

Configuration file for `Export` service can be sent over MQTT using [Agent][agent] service.

```
mosquitto_pub -u <thing_id> -P <thing_key> -t channels/<control_ch_id>/messages/req -h localhost -p 18831  -m  "[{\"bn\":\"1:\", \"n\":\"config\", \"vs\":\"save, export, <config_file_path>, <file_content_base64>\"}]"
```

`vs="save, export, config_file_path, file_content_base64"` - vs determines where to save file and contains file content in base64 encoding payload:
```
b,_ := toml.Marshal(export.Config)
payload := base64.StdEncoding.EncodeToString(b)
```

### Using configure script

There is a `configuration.sh` script in a `scripts` directory that can be used for automatic configuration and start up of remotely deployed `export`. For this to work it is presumed that `mainflux-export` and `scripts/export_start` are placed in executable path on remote device. Additionally this script requires that remote device is provisioned following the steps described for [provision](provision.md) service.

To run it first edit script to set parameters
```
MTLS=false
EXTERNAL_KEY='raspberry'
EXTERNAL_ID='pi'
MAINFLUX_HOST='mainflux.com'
MAINFLUX_USER_EMAIL='edge@email.com'
MAINFLUX_USER_PASSWORD='12345678'
```

`EXTERNAL_KEY` and `EXTERNAL_ID` are parameters posted to `/mapping` endpoint of `provision` service, `MAINFLUX_HOST` is location of cloud instance of Mainflux that `export` should connect to and `MAINFLUX_USER_EMAIL` and `MAINFLUX_USER_PASSWORD` are users credentials in the cloud.

[conftoml](https://github.com/mainflux/export/blob/master/configs/config.toml)
[docker-compose](https://github.com/mainflux/mainflux/docker/docker-compose.yml)
[env](https://github.com/mainflux/export#environmet-variables)
[agent](https://github.com/mainflux/agent)

## Example deployment

### Edge deployment

The following are steps that are an example usage of Mainflux components to connect edge with cloud.
We will start Mainflux in the cloud with additional services [Bootstrap](bootstrap) and [Provision](provision).
Using [Bootstrap](bootstrap) and [Provision](provision) we will create a configuration for use in gateway deployment.
On the gateway we will start services [Agent](/edge/#agent) and [Export](/edge/#export) using previously created configuration.

## Services in the cloud
Start the Mainflux:

```bash
docker-compose -f docker/docker-compose.yml up
```

Start the Bootstrap service:

```bash
docker-compose -f docker/addons/bootstrap/docker-compose.yml up
```

Start the Provision service

```bash
docker-compose -f docker/addons/provision/docker-compose.yml up
```

Create user:
```bash
mainflux-cli -m http://localhost:8180 users create test@email.com 12345678
```

Obtain user token:
```bash
mainflux-cli -m http://localhost:8180 users token test@email.com 12345678

created: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1ODk5MDQ4MDQsImlhdCI6MTU4OTg2ODgwNCwiaXNzIjoibWFpbmZsdXguYXV0aG4iLCJzdWIiOiJ0ZXN0QGVtYWlsLmNvbSIsInR5cGUiOjB9.VSwpGoflOLqrHlCGoVVFPBdnnvsAhv2gc3EomXg9yM0

TOK=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1ODk5MDQ4MDQsImlhdCI6MTU4OTg2ODgwNCwiaXNzIjoibWFpbmZsdXguYXV0aG4iLCJzdWIiOiJ0ZXN0QGVtYWlsLmNvbSIsInR5cGUiOjB9.VSwpGoflOLqrHlCGoVVFPBdnnvsAhv2gc3EomXg9yM0
```

Provision a gateway:

```bash
curl -s -S  -X POST  http://localhost:8190/mapping -H "Authorization: $TOK" -H 'Content-Type: application/json'   -d '{"name":"testing",  "external_id" : "54:FG:66:DC:43", "external_key":"223334fw2" }' | jq
```
```json
{
  "things": [
    {
      "id": "88529fb2-6c1e-4b60-b9ab-73b5d89f7404",
      "name": "thing",
      "key": "3529c1bb-7211-4d40-9cd8-b05833196093",
      "metadata": {
        "external_id": "54:FG:66:DC:43"
      }
    }
  ],
  "channels": [
    {
      "id": "1aa3f736-0bd3-44b5-a917-a72cc743f633",
      "name": "control-channel",
      "metadata": {
        "type": "control"
      }
    },
    {
      "id": "e2adcfa6-96b2-425d-8cd4-ff8cb9c056ce",
      "name": "data-channel",
      "metadata": {
        "type": "data"
      }
    }
  ],
  "whitelisted": {
    "88529fb2-6c1e-4b60-b9ab-73b5d89f7404": true
  }
}
```

Parameters <external_id> and <external_key> are representing the gateway. `Provision` will use them to create a bootstrap configuration that will make a relation with Mainflux entities used for connection, authentication and authorization `thing` and `channel`.
These parameters will be used by `Agent` service on the gateway to retrieve that information and establish a connection with the cloud.

## Services on the Edge

### Agent

Start the [NATS][nats] and [Agent][agent] service:
```bash
gnatsd
MF_AGENT_BOOTSTRAP_ID=54:FG:66:DC:43 \
MF_AGENT_BOOTSTRAP_KEY="223334fw2" \
MF_AGENT_BOOTSTRAP_URL=http://localhost:8202/things/bootstrap \
build/mainflux-agent
{"level":"info","message":"Requesting config for 54:FG:66:DC:43 from http://localhost:8202/things/bootstrap","ts":"2020-05-07T15:50:58.041145096Z"}
{"level":"info","message":"Getting config for 54:FG:66:DC:43 from http://localhost:8202/things/bootstrap succeeded","ts":"2020-05-07T15:50:58.120779415Z"}
{"level":"info","message":"Saving export config file /configs/export/config.toml","ts":"2020-05-07T15:50:58.121602229Z"}
{"level":"warn","message":"Failed to save export config file Error writing config file: open /configs/export/config.toml: no such file or directory","ts":"2020-05-07T15:50:58.121752142Z"}
{"level":"info","message":"Client agent-88529fb2-6c1e-4b60-b9ab-73b5d89f7404 connected","ts":"2020-05-07T15:50:58.128500603Z"}
{"level":"info","message":"Agent service started, exposed port 9003","ts":"2020-05-07T15:50:58.128531057Z"}
```

### Export

```
git clone https://github.com/mainflux/export
make
```
Edit the `configs/config.toml` setting 
- `username` - thing from the results of provision request.
- `password` - key from the results of provision request.
- `mqtt_topic` - in routes set to `channels/<channel_data_id>/messages` from results of provision.
- `nats_topic` - whatever you need, export will subscribe to `export.<nats_topic>` and forward messages to MQTT.
- `host` - url of MQTT broker.
  
```toml
[exp]
  cache_pass = ""
  cache_url = ""
  log_level = "debug"
  nats = "localhost:4222"
  port = "8170"

[mqtt]
  ca_path = ""
  cert_path = ""
  host = "tcp://localhost:1883"
  mtls = false
  password = "3529c1bb-7211-4d40-9cd8-b05833196093"
  priv_key_path = ""
  qos = 0
  retain = false
  skip_tls_ver = false
  username = "88529fb2-6c1e-4b60-b9ab-73b5d89f7404"

[[routes]]
  mqtt_topic = "channels/e2adcfa6-96b2-425d-8cd4-ff8cb9c056ce/messages"
  nats_topic = ">"
  workers = 10
```


```bash
cd build
./mainflux-export
2020/05/07 17:36:57 Configuration loaded from file ../configs/config.toml
{"level":"info","message":"Export service started, exposed port :8170","ts":"2020-05-07T15:36:57.528398548Z"}
{"level":"debug","message":"Client export-88529fb2-6c1e-4b60-b9ab-73b5d89f7404 connected","ts":"2020-05-07T15:36:57.528405818Z"}
```

#### Testing Export
```bash
git clone https://github.com/nats-io/nats.go
cd github.com/nats-io/nats.go/examples/nats-pub
go run main.go -s http://localhost:4222 export.test "[{\"bn\":\"test\"}]";
```

We have configured route for export, `nats_topic = ">"` means that it will listen to `NATS` subject `export.>` and `mqtt_topic` is configured so that data will be sent to MQTT broker on topic `channels/e2adcfa6-96b2-425d-8cd4-ff8cb9c056ce/messages` with appended `NATS` subject.

In terminal where export is started you should see following message:
```log
{"level":"debug","message":"Published to: export.test, payload: [{\"bn\":\"test\"}]","ts":"2020-05-08T15:14:15.757298992Z"}
```

In Mainflux `mqtt` service:
```log
mainflux-mqtt   | {"level":"info","message":"Publish - client ID export-88529fb2-6c1e-4b60-b9ab-73b5d89f7404 to the topic: channels/e2adcfa6-96b2-425d-8cd4-ff8cb9c056ce/messages/export/test","ts":"2020-05-08T15:16:02.999684791Z"}
```
